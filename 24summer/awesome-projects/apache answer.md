[docs](https://answer.apache.org/docs)
[githuh](https://github.com/apache/incubator-answer)

# Overview
- 前后端分离, 架构非常清晰
- 整个项目非常庞大, go 源码就有 6w 行, 模块和功能非常多

# Dependency Injection
- 模块区分非常清晰, 依赖关系很多, 使用了 `wire` 来管理依赖, 并且使用依赖注入非常多, 学习下分模块和依赖注入方法

### wire
- `wire` 的核心概念是 `provider+injector`
- `provider`: 所有新建被依赖项的构造函数, 一般是 `func New...()`
- `injector`: 就是将所有层层依赖组合产出最后结果的那个函数, 一般是 `func init...(...) { wire.Build(...) }`
- 从大层面上来说, 注入的依赖关系可以这样划分
	- `NewApplication(server)app`
	- `NewServer(router)server`
	- `NewRouter(controller)router` (handler???)
	- `NewController(service)controller`
	- `NewService(repo)service`
	- `NewRepo()repo`
- 当然还有很多中间件, 渲染逻辑也在不同的地方被依赖, 也有少量的跨级依赖和同级依赖, 但是最主要的分层逻辑就是这样做的
- 使用过程, 在依赖项很多时, 将多个相关联的, 或者同属于一个 `package` 的依赖构造函数打包到同一个 `provider set` 中, 向上传递给 `wire.Build()` 时只需要提供这个 `provider set` 即可
- 最底层的依赖是 `config, log` 这两部分由 main 管理, 然后作为参数传递给 wire


```go
// initApplication init application.
func initApplication(
	debug bool,
	serverConf *conf.Server,
	dbConf *data.Database,
	cacheConf *data.CacheConf,
	i18nConf *translator.I18n,
	swaggerConf *router.SwaggerConfig,
	serviceConf *service_config.ServiceConfig,
	uiConf *server.UI,
	logConf log.Logger) (*pacman.Application, func(), error) {
	panic(wire.Build(
		server.ProviderSetServer,
		router.ProviderSetRouter,
		controller.ProviderSetController,
		controller_admin.ProviderSetController,
		templaterender.ProviderSetTemplateRenderController,
		service.ProviderSetService,
		cron.ProviderSetService,
		repo.ProviderSetRepo,
		translator.ProviderSet,
		middleware.ProviderSetMiddleware,
		newApplication,
	))
}


// ProviderSetRouter is providers.
var ProviderSetRouter = wire.NewSet(
	NewAnswerAPIRouter,
	NewSwaggerRouter,
	NewStaticRouter,
	NewUIRouter,
	NewTemplateRouter,
	NewPluginAPIRouter,
)

```


# Commands
- `answer` 同时还是一个 `cobra-cli` 应用, 支持很多命令行子
	- `init`
	- `run`
	- `upgrade`
	- ...


# Design Architecture
- 这里都是同步的 api, 用了缓存+主数据库 `go-cache + sqlite`
- 中间件


# Software Architecture
- `server`
- `controller`
- `service`
- `repo`

### Question
- 以 `question` 模块为例, 这里详细地说明 `controller-service-repo` 的架构是如何实现的
- `QuestionController` 作为依赖注入到 `NewServer()` 中, 然后利用 `QuestionController` 的方法来处理路由

```go
// QuestionController question controller
type QuestionController struct {
	questionService     *content.QuestionService
	answerService       *content.AnswerService
	rankService         *rank.RankService
	siteInfoService     siteinfo_common.SiteInfoCommonService
	actionService       *action.CaptchaService
	rateLimitMiddleware *middleware.RateLimitMiddleware
}

// NewQuestionController new controller
func NewQuestionController(
	questionService *content.QuestionService,
	answerService *content.AnswerService,
	rankService *rank.RankService,
	siteInfoService siteinfo_common.SiteInfoCommonService,
	actionService *action.CaptchaService,
	rateLimitMiddleware *middleware.RateLimitMiddleware,
) *QuestionController {
	return &QuestionController{
		questionService:     questionService,
		answerService:       answerService,
		rankService:         rankService,
		siteInfoService:     siteInfoService,
		actionService:       actionService,
		rateLimitMiddleware: rateLimitMiddleware,
	}
}
```
- `QuestionController` 依赖于这些 `service, middleware`, 同样使用依赖注入创建, 具体实现结构是指针, 接口则是接口类型


![[Pasted image 20240815180802.png]]
- 一些关于 `question` 模块的接口, 这些接口的路由最后的处理函数都是 `QuestionController` 的方法 (接入层)


- 以 `QuestionController.AddQuestion()` 为例, 看看 `controller` 是如何跟 `service` 进行交互的
	1. 绑定校验请求参数到 `schema.AddQuestionReq`
	2. 限流中间件进行重复请求限流处理
	3. `tagService` 检查抽取出本次新建 `question` 是否附带着新建 `tag`
	4. `questionService` 检查问题是否重复, 是否可新增问题
	5. `questionService.AddQuestion(...) schema.AddQuestionResp` 新增问题
	6. 返回响应

```go
// AddQuestion add question
// @Summary add question
// @Description add question
// @Tags Question
// @Accept json
// @Produce json
// @Security ApiKeyAuth
// @Param data body schema.QuestionAdd true "question"
// @Success 200 {object} handler.RespBody
// @Router /answer/api/v1/question [post]
func (qc *QuestionController) AddQuestion(ctx *gin.Context) {
	req := &schema.QuestionAdd{}
	errFields := handler.BindAndCheckReturnErr(ctx, req)
	if ctx.IsAborted() {
		return
	}
	reject, rejectKey := qc.rateLimitMiddleware.DuplicateRequestRejection(ctx, req)
	if reject {
		return
	}
	defer func() {
		// If status is not 200 means that the bad request has been returned, so the record should be cleared
		if ctx.Writer.Status() != http.StatusOK {
			qc.rateLimitMiddleware.DuplicateRequestClear(ctx, rejectKey)
		}
	}()

	req.UserID = middleware.GetLoginUserIDFromContext(ctx)
	canList, requireRanks, err := qc.rankService.CheckOperationPermissionsForRanks(ctx, req.UserID, []string{
		permission.QuestionAdd,
		permission.QuestionEdit,
		permission.QuestionDelete,
		permission.QuestionClose,
		permission.QuestionReopen,
		permission.TagUseReservedTag,
		permission.TagAdd,
		permission.LinkUrlLimit,
	})
	if err != nil {
		handler.HandleResponse(ctx, err, nil)
		return
	}
	linkUrlLimitUser := canList[7]
	isAdmin := middleware.GetUserIsAdminModerator(ctx)
	if !isAdmin || !linkUrlLimitUser {
		captchaPass := qc.actionService.ActionRecordVerifyCaptcha(ctx, entity.CaptchaActionQuestion, req.UserID, req.CaptchaID, req.CaptchaCode)
		if !captchaPass {
			errFields := append([]*validator.FormErrorField{}, &validator.FormErrorField{
				ErrorField: "captcha_code",
				ErrorMsg:   translator.Tr(handler.GetLang(ctx), reason.CaptchaVerificationFailed),
			})
			handler.HandleResponse(ctx, errors.BadRequest(reason.CaptchaVerificationFailed), errFields)
			return
		}
	}

	req.CanAdd = canList[0]
	req.CanEdit = canList[1]
	req.CanDelete = canList[2]
	req.CanClose = canList[3]
	req.CanReopen = canList[4]
	req.CanUseReservedTag = canList[5]
	req.CanAddTag = canList[6]
	if !req.CanAdd {
		handler.HandleResponse(ctx, errors.Forbidden(reason.RankFailToMeetTheCondition), nil)
		return
	}

	// can add tag
	hasNewTag, err := qc.questionService.HasNewTag(ctx, req.Tags)
	if err != nil {
		handler.HandleResponse(ctx, err, nil)
		return
	}
	if !req.CanAddTag && hasNewTag {
		lang := handler.GetLang(ctx)
		msg := translator.TrWithData(lang, reason.NoEnoughRankToOperate, &schema.PermissionTrTplData{Rank: requireRanks[6]})
		handler.HandleResponse(ctx, errors.Forbidden(reason.NoEnoughRankToOperate).WithMsg(msg), nil)
		return
	}

	errList, err := qc.questionService.CheckAddQuestion(ctx, req)
	if err != nil {
		errlist, ok := errList.([]*validator.FormErrorField)
		if ok {
			errFields = append(errFields, errlist...)
		}
	}

	if len(errFields) > 0 {
		handler.HandleResponse(ctx, errors.BadRequest(reason.RequestFormatError), errFields)
		return
	}

	req.UserAgent = ctx.GetHeader("User-Agent")
	req.IP = ctx.ClientIP()

	resp, err := qc.questionService.AddQuestion(ctx, req)
	if err != nil {
		errlist, ok := resp.([]*validator.FormErrorField)
		if ok {
			errFields = append(errFields, errlist...)
		}
	}

	if len(errFields) > 0 {
		handler.HandleResponse(ctx, errors.BadRequest(reason.RequestFormatError), errFields)
		return
	}
	if !isAdmin || !linkUrlLimitUser {
		qc.actionService.ActionRecordAdd(ctx, entity.CaptchaActionQuestion, req.UserID)
	}
	handler.HandleResponse(ctx, err, resp)
}
```


- `service` 层, 具体的业务逻辑在这里实现, 虽然说业务逻辑在这里实现, 但是实际上 `controller` 中调用很多 `service, middleware` 应该也算是业务逻辑, 因此应该是原子业务逻辑在这里实现
	1. ... 处理 tag, recommend 之类的东西
	2. 从 `schema.AddQuestionReq` 中组合参数, 得到与数据库交互的 `entity.Question`
	3. `questionRepo.AddQuestion(question)` 通过 `repo` 与数据库进行交互
	4. ...

```go
// QuestionService user service
type QuestionService struct {
	questionRepo                     questioncommon.QuestionRepo
	answerRepo                       answercommon.AnswerRepo
	tagCommon                        *tagcommon.TagCommonService
	questioncommon                   *questioncommon.QuestionCommon
	userCommon                       *usercommon.UserCommon
	userRepo                         usercommon.UserRepo
	userRoleRelService               *role.UserRoleRelService
	revisionService                  *revision_common.RevisionService
	metaService                      *metacommon.MetaCommonService
	collectionCommon                 *collectioncommon.CollectionCommon
	answerActivityService            *activity.AnswerActivityService
	emailService                     *export.EmailService
	notificationQueueService         notice_queue.NotificationQueueService
	externalNotificationQueueService notice_queue.ExternalNotificationQueueService
	activityQueueService             activity_queue.ActivityQueueService
	siteInfoService                  siteinfo_common.SiteInfoCommonService
	newQuestionNotificationService   *notification.ExternalNotificationService
	reviewService                    *review.ReviewService
	configService                    *config.ConfigService
}

// AddQuestion add question
func (qs *QuestionService) AddQuestion(ctx context.Context, req *schema.QuestionAdd) (questionInfo any, err error) {
	// ...

	question := &entity.Question{}
	now := time.Now()
	question.UserID = req.UserID
	question.Title = req.Title
	question.OriginalText = req.Content
	question.ParsedText = req.HTML
	question.AcceptedAnswerID = "0"
	question.LastAnswerID = "0"
	question.LastEditUserID = "0"
	//question.PostUpdateTime = nil
	question.Status = entity.QuestionStatusPending
	question.RevisionID = "0"
	question.CreatedAt = now
	question.PostUpdateTime = now
	question.Pin = entity.QuestionUnPin
	question.Show = entity.QuestionShow
	//question.UpdatedAt = nil
	err = qs.questionRepo.AddQuestion(ctx, question)
	if err != nil {
		return
	}
	question.Status = qs.reviewService.AddQuestionReview(ctx, question, req.Tags, req.IP, req.UserAgent)
	if err := qs.questionRepo.UpdateQuestionStatus(ctx, question.ID, question.Status); err != nil {
		return nil, err
	}
	objectTagData := schema.TagChange{}
	objectTagData.ObjectID = question.ID
	objectTagData.Tags = req.Tags
	objectTagData.UserID = req.UserID
	err = qs.ChangeTag(ctx, &objectTagData)
	if err != nil {
		return
	}
	_ = qs.questionRepo.UpdateSearch(ctx, question.ID)

	revisionDTO := &schema.AddRevisionDTO{
		UserID:   question.UserID,
		ObjectID: question.ID,
		Title:    question.Title,
	}

	questionWithTagsRevision, err := qs.changeQuestionToRevision(ctx, question, tags)
	if err != nil {
		return nil, err
	}
	infoJSON, _ := json.Marshal(questionWithTagsRevision)
	revisionDTO.Content = string(infoJSON)
	revisionID, err := qs.revisionService.AddRevision(ctx, revisionDTO, true)
	if err != nil {
		return
	}

	// user add question count
	userQuestionCount, err := qs.questioncommon.GetUserQuestionCount(ctx, question.UserID)
	if err != nil {
		log.Errorf("get user question count error %v", err)
	} else {
		err = qs.userCommon.UpdateQuestionCount(ctx, question.UserID, userQuestionCount)
		if err != nil {
			log.Errorf("update user question count error %v", err)
		}
	}

	qs.activityQueueService.Send(ctx, &schema.ActivityMsg{
		UserID:           question.UserID,
		ObjectID:         question.ID,
		OriginalObjectID: question.ID,
		ActivityTypeKey:  constant.ActQuestionAsked,
		RevisionID:       revisionID,
	})

	if question.Status == entity.QuestionStatusAvailable {
		qs.externalNotificationQueueService.Send(ctx,
			schema.CreateNewQuestionNotificationMsg(question.ID, question.Title, question.UserID, tags))
	}

	questionInfo, err = qs.GetQuestion(ctx, question.ID, question.UserID, req.QuestionPermission)
	return
}
```

- `repo` 封装了直接跟数据库交互的逻辑, 这里有些特殊, 使用一个自增表来做 unique, 所以还依赖了另外一个 `uniqueIDRepo`
	1. 获取 `uniqueID`
	2. 插入数据

```go
// questionRepo question repository
type questionRepo struct {
	data         *data.Data
	uniqueIDRepo unique.UniqueIDRepo
}

// AddQuestion add question
func (qr *questionRepo) AddQuestion(ctx context.Context, question *entity.Question) (err error) {
	question.ID, err = qr.uniqueIDRepo.GenUniqueIDStr(ctx, question.TableName())
	if err != nil {
		return errors.InternalServer(reason.DatabaseError).WithError(err).WithStack()
	}
	_, err = qr.data.DB.Context(ctx).Insert(question)
	if err != nil {
		return errors.InternalServer(reason.DatabaseError).WithError(err).WithStack()
	}
	if handler.GetEnableShortID(ctx) {
		question.ID = uid.EnShortID(question.ID)
	}
	return
}
```


# Caching
- 使用了 `go-cache` 作为缓存, 没有用外部组件, 直接使用库了
- 数据类型的使用
	- `string`
	- `int64`
- 使用这些接口来处理缓存 `data.Cache.GetString(...), data.Cache.SetString(...), data.Cache.GetInt64(...), data.Cache.SetInt64(...), data.Cache.Del(...)`

### 使用 `string` 缓存的例子:
- `userInfo` 被高强度的缓存, 有两种方式 `UserTokenCacheKey+accessToken or UserStatusChangedCacheKey+userID ==> userInfo`
	- 中间件和 controller 调用 `AuthService.GetUserCacheInfo(..)` 获取缓存的 userinfo
	- `AuthService.GetUserCacheInfo(accessToken string)`
	- `AuthRepo.GetUserCacheInfo(accessToken string)` 获取 token cache userinfo
	- `AuthRepo.GetUserStatus(userID)` 获取 status cache userinfo, 与 token cache 对比更新 token cache (这里应该是因为在登录之后, 可能用户的状态发生了改变, 只会修改 status cache, 因此 status cache 往往都是更新的, 并借此机会更新一下 token cache)
- `accessToken` 也被缓存 `UserVisitTokenCacheKey+visitToken ==> accessToken`
- `tokenMapping`
- `config`



# Testing



