# Nest项目总结

该项目是一个使用Node.JS的NestJS框架开发的中转站点项目，开发语言是Typescript，使用Docker部署，作为前后端分离项目中的前端页面和微服务接口之前的桥梁。

## 项目功能

1.  接口中转：为外部项目提供接口中转服务并且支持原始接口结果转换，如XML --> Json 
2.  前端页面部署：前后端分离项目前端页面可以放在服务器中部署
3.  权限验证：前端站点的登录验证以及部分集团的权限验证会在请求原始接口之前通过中间件的形式完成

## 源代码说明

NestJS是一个类Spring的框架，内部内置MVC和容器，因此使用过Java或者.Net的同学可以快速上手，因此代码也很容易理解。

### 接口中转

首先来说接口中转的源代码。接口中转的逻辑其实就是在项目中通过传入参数请求原始接口，记录日志然后将结果返回的流程。这个流程很简单，以GET请求为例，源代码如下：

``` Typescript
Controller:
@Get(':serviceName')
public async transferGetRequest(@Request() req, @Param() param, @Response() res) {
    const transferResultModel = await this.transferService.getServiceTransfer(req, param);
    // 前端页面不返回bid等参数
    // res.header(validateResultModel.data.headerData);
    let contentType = req.headers['content-type'];
    if (!contentType) {
        contentType = 'application/json; charset=UTF-8';
    }
    res.header('content-type', contentType);
    // res.header("Access-Control-Allow-Origin", "*");
    if (transferResultModel.code === HttpStatus.OK) {
        const changtype = req.query.changetype || '';
        const result = changtype === 'json' ? this.commonService.xmlTojson(transferResultModel.data) : transferResultModel.data;
        res.status(transferResultModel.code).json(result);
    } else {
        res.status(transferResultModel.code).json(transferResultModel.data);
    }
    return;
}

Service:
public async getServiceTransfer(req: any, param: any): Promise<CommonResponseModel> {
    const validateResultModel = this.paramValidate(req, param);
    if (validateResultModel.code !== HttpStatus.OK) {
        return validateResultModel;
    }

    const resultModel = new CommonResponseModel();
    try {
        const invokeParam = this.getReqQueryParam(req);
        // tslint:disable-next-line: max-line-length
        const result = await this.invokeService.invokeGet(param.serviceName, req, validateResultModel.data.serviceUrl, invokeParam, validateResultModel.data.headerData);
        if (!result) {
            resultModel.code = HttpStatus.INTERNAL_SERVER_ERROR;
            resultModel.message = '服务接口请求失败';
            return resultModel;
        } else {
            resultModel.code = HttpStatus.OK;
            resultModel.data = result.data;
            resultModel.message = '请求成功';
            return resultModel;
        }
    } catch (exception) {
        resultModel.code = HttpStatus.INTERNAL_SERVER_ERROR;
        resultModel.message = '服务接口请求失败 --> ' + exception;
        return resultModel;
    }
}
```

因此作为中转的代码很简单，但是有一点需要说明，就是我把中转接口的配置作为了路由的一部分，即：假如项目地址为：http://127.0.0.1:3000/api/{configKey}，那么其中的configKey即为接口的配置key，一个key对应着一个配置文件中的接口。

这样做的好处在于，我可以通过路由来进行权限验证，例如：如果是B端权限验证，我的路由地址是http://127.0.0.1:3000/api/{configKey}，如果是服务云权限验证，我可以的路由为http://127.0.0.1:3000/api/{configKey}。可以很好地利用中间件的特性。

此外，有些原始接口返回的结果是`XML`格式的，这种格式目前的主流前端框架都不使用了，因此内部内置了一个`XML` --> `Json`的转换器，具体代码如下：

``` Typescript
if (transferResultModel.code === HttpStatus.OK) {
    const changtype = req.query.changetype || '';
    const result = changtype === 'json' ? this.commonService.xmlTojson(transferResultModel.data) : transferResultModel.data;
    // ...
}

public xmlTojson(str: string): string {
    let resultJson = str;
    try {
        if (!this.IsJsonString(str)) {
            // tslint:disable-next-line: only-arrow-functions
            xmlParser.parseString(str, function(err, result) {
                if (err) {
                    return str;
                }
                // 将返回的结果再次格式化
                resultJson = result;
            });
        }
    } catch (ex) {
        resultJson = str;
    }
    return resultJson;
}
```

其中格式转换使用到了`xml2js`JS库，代码封装在了`CommonService`中。

### 前端页面部署

我们这边的前端页面使用Vue开发，因此前端的所有逻辑都自包含在了一个html文件中，这个中转站点就仅仅做了一个部署那个html文件的工作，因此源代码也很简单，就是请求资源中的html文件即可：

``` Typescript
Controller:
@Get(':viewName')
public router(@Response() res, @Param() param) {
    const viewPath = this.vuerouterService.getViewDir(param.viewName as string);
    try {
        if (viewPath) {
            res.sendFile(viewPath);
        } else {
            res.status(HttpStatus.BAD_REQUEST).json({
                code: HttpStatus.BAD_REQUEST,
                message: '没有对应视图',
            });
        }
    } catch {
        const result = new CommonResponseModel();
        result.code = HttpStatus.BAD_REQUEST;
        result.message = '非法路由地址';
        return result;
    }
}

Service:
public getViewDir(viewName: string): string {
    const splitArr = viewName.split('.');
    if (splitArr && splitArr.length > 1) {
        return join(__dirname, '../../views', viewName);
    } else {
        return join(__dirname, '../../views', `${ viewName }.html`);
    }
}
```

这段代码的意思就是找到我们配置中的views路径下对应名称为viewName的文件并且请求。其中的viewName和上述中的原始接口也一样，也是参数化在路由中的。例请求地址：http://127.0.0.1:3000/views/index.html 就可以访问前端页面了。

### 权限验证

权限验证其实是在调用端请求原始接口之前做的一套登录信息验证等操作，如果检测到用户未登录，则不会请求原始接口直接返回403错误。因为部分原始接口需要用到B端登录信息中的一些数据，这部分逻辑放在了中转中。

这部分逻辑我把它写在了中间件中，`中间件大概就类似于一个拦截器`，可以针对不同的`路由`来处理请求MVC之前的一些逻辑。

由于家居这边需要两种不同的权限集，一个是B端权限验证，一个是服务云权限验证，两者无交集。因此我们需要完成两个不同的中间件，并且为不同的路由分别配置不同的中间件。

中间件配置的源代码如下：

``` Typescript
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer.apply(AuthMiddleware).forRoutes('/api');
    consumer.apply(AuthMiddleware).forRoutes('/wap');
    consumer.apply(OAMiddleware).forRoutes('/oa');
    consumer.apply(OAMiddleware).forRoutes('/oaauth');
  }
}
```

这部分代码在基础module的`app.module.ts`中，这个文件作为整个程序的入口加载器来使用，类似于DotNet Core框架中的`Startup`，因此中间件的配置在此处完成。

可以从源码看到，我针对`api`和`wap`路由使用了`AuthMiddleware（B端组织关系权限）`中间件，为`oa`和`oaauth`路由使用了`OAMiddleware（服务云权限）`中间件。

至于中间件的编写，其实逻辑就是请求对应权限的接口查看结果罢了，其实没有什么逻辑，代码我贴一个B端组织关系的作为示例：

``` Typescript
@Injectable()
export class AuthMiddleware implements NestMiddleware {
  // tslint:disable-next-line: max-line-length
  constructor(private readonly cacheService: CacheService, private readonly invokeService: InvokeService, private readonly configuration: ConfigurationService) {}
  resolve(...args: any[]): MiddlewareFunction {
    return async (req, res, next) => {
      const responseData = new CommonResponseModel();
      try {
        const sfyt = req.cookies.sfyt;
        if (!sfyt) {
          responseData.code = HttpStatus.UNAUTHORIZED;
          responseData.data = null;
          responseData.message = '用户未登录';
          res.status(HttpStatus.UNAUTHORIZED).json(responseData);
          return;
        } else {
          let cacheValue = await this.cacheService.get<any>(sfyt);
          /*如果没有从缓存读取到，则直接查询接口*/
          if (!cacheValue) {
            const sfut = req.cookies.sfut;
            if (!sfut) {
              responseData.code = HttpStatus.UNAUTHORIZED;
              responseData.data = null;
              responseData.message = '用户未登录';
              res.status(HttpStatus.UNAUTHORIZED).json(responseData);
              return;
            } else {
              // 请求B端组织关系接口
              if (!invokeResult) {
                responseData.code = HttpStatus.SERVICE_UNAVAILABLE;
                responseData.data = null;
                responseData.message = '用户登录接口请求失败';
                res.status(HttpStatus.SERVICE_UNAVAILABLE).json(responseData);
                return;
              } else {
                const result = invokeResult.data;
                if (result.code != 1) {
                  responseData.code = HttpStatus.SERVICE_UNAVAILABLE;
                  responseData.data = result.meessage;
                  responseData.message = '用户登录接口请求失败';
                  res.status(HttpStatus.SERVICE_UNAVAILABLE).json(responseData);
                  return;
                } else {
                  req.loginData = result.data;
                  req.authType = AuthTypeEnum.BSide;
                  cacheValue = JSON.stringify(result.data);
                  await this.cacheService.set(sfyt, cacheValue);
                  next();
                }
              }
            }
          } else {
            req.loginData = JSON.parse(cacheValue);
            req.authType = AuthTypeEnum.BSide;
            next();
          }
        }
      } catch {
        responseData.code = HttpStatus.INTERNAL_SERVER_ERROR;
        responseData.message = '服务器错误';
        res.status(HttpStatus.INTERNAL_SERVER_ERROR).json(responseData);
        return;
      }
    };
  }
}
```

中间件有自己固定的语法，我们只需要完成内部的接口请求逻辑即可。

另外，内部使用到了Redis缓存，NestJS项目使用Redis还比较麻烦，使用原生的JS库会有问题，因此我在网上找了很多代码最终封装了一个Redis请求的Service，有兴趣可以看`CacheService`。

## 部署

目前项目是使用`Docker`部署的，后期中间机房的`K8S`搭建成功之后会上线`K8S`，其中`Dockerfile`如下：

``` Dockerfile
FROM dockerhub.3g.fang.com/base/centos7.6/nodejs/nodejs-10.15-image:v1

RUN npm install -g pm2

COPY --chown=webservice:webservice ./package*.json ./

RUN npm install --production

COPY --chown=webservice:webservice ./ ./

RUN npm run build

COPY --chown=webservice:webservice ./src/views ./dist/views

EXPOSE 3000

CMD [ "pm2-runtime", "start", "npm", "--", "start" ]
```

从`Dockerfile`中可以看到，基础镜像是公司的Nodejs镜像，内部内置了一个`pm2`的管理程序，用来内置启动项目，并且遵循公司要求的权限控制，对外暴露`3000端口`。

项目启动脚本都写在了`package.json`中，有兴趣可以查看。

## 其他权限接入

其他权限验证接入其实只需要再单独编写对应的权限验证中间件，然后配置到对应的路由中即可，这些流程在`权限验证`中已经说到了再次就不再详述。

## 未来的改善

这个项目还是有很多可以改善的点的，在此我具体说明两个我觉得比较重要的点。

### 中转配置文件分组问题

目前的所有配置文件都是写在一个配置中的，如`service-dev.config.ts`，这里的每一个key就对应了一个中转接口。

光从文件中我们就能发现存在两个严重的问题：

1. 该配置文件是`.ts`格式的，因此每次修改配置文件需要重新编译
2. 该配置文件的key是一一对应的，就导致了每添加一个接口就需要改一遍配置，但是对于很多接口其实根域名是一样的，只是后面的路由地址不同，在此就需要配置多个key，非常繁琐并且后期配置文件庞大无法管理

对于这两个问题，第一个目前暂时没有直接的解决方案，因为就算讲ts文件改为文件这个问题还是存在，需要重启才能读取，目前尚不知道站点有没有热启动的方式没有深入研究，但是随着第二个问题得深入可以解决这个问题。

第二个问题是可以改善的，就是标题所说的配置分组，可以将配置进行分组，每个组对应一个key，将请求路由作为三级，这样就可以完成同一个根域名下的不同配置不用增加在一个配置文件中。

并且，既然有组的概念，就应该存入数据库，每次站点启动读入内存，在内存中完善。

而如果作的更好一点，可以使用类似计时器的功能定时从数据库中拉取一次配置替换内存中的旧配置，这样就可以解决第一个问题中没有热启动的问题。

### Linux镜像制作无权限问题

这个项目有个奇怪的点是，Linux和Windows的Docker所制作的镜像中的文件权限不一样。Windows下制作的镜像，内部的文件中的文件都是绿色的可执行文件，但是Linux的Docker制作完就是白色的纯文件。出文件无法使用编译的方式直接运行，就导致了之前有一段时间Jenkins打包制作的镜像无法使用。

这个问题之后改为生产运行暂时解决了，但是这样解决埋下的伏笔在于无法挂载静态页面了，因为内部没有权限无法通过路径来找对应的挂载文件了，所以每次传前端页面都需要制作镜像。

这也是一个问题，但是之后考虑了一下，这个问题可以在日后上线K8S解决。因为K8S无法挂载文件，每次上传都需要制作镜像，这个问题就被动地解决了。