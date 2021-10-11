如何在AWS Location上增加一个标识点？

How to add an identification point to AWS Location?



**我想实现下图功能，请问使用AWS Location怎么做？**

![image-20211011171442841](https://raw.githubusercontent.com/liangyimingcom/storage/master/PicGo/image-20211011171442841.png)



回答预热：

- 这个功能其实和AWS Location关系不大，是调用一个地图库maplibregl；
- 调用一个地图库maplibregl的API，在地图上画上一个标记，就算不用Location，也能画；
- 我猜 调用一个地图库maplibregl的API 是一个非常基本的功能，凡是开发地图应用的人应该都知道；
- 所以才没有在AWS库里面写一个独立的sample 😭
- 地图库maplibregl的API 也找到了，让用户参考 https://maplibre.org/maplibre-gl-js-docs/example/add-a-marker/



答案：

这个功能叫：Add a marker to the map，请参考https://aws.amazon.com/blogs/mobile/add-a-map-to-your-webpage-with-amazon-location-service/

``` javascript
const marker = new maplibregl.Marker().setLngLat(position).addTo(map);
```



![image-20211011171737625](https://raw.githubusercontent.com/liangyimingcom/storage/master/PicGo/image-20211011171737625.png)



背景：客户需求使用LocationService获取其用户位置，并在移动设备上显示出地址及标识其位置
使用服务为 LocationService中的Place.index，但通过接口调用后，测试后发现只能返回其指定的坐标信息，但无法在显示的地图上标识位置，客户的想法是需要显示出位置，就像LocationService中的Expore那样；



**：：：中文翻译如下：：：**

------



# 使用 Amazon Location Service 将地图添加到您的网页



### *本文由 Amazon Location Service 首席工程师 Seth Fitzsimmons 和高级产品经理 Andrew Johnson 撰写*

Web 开发的一个常见用例是将地图添加到带有特定位置（例如办公室或零售店）上的标记的网页。Amazon Location 是一个很好的解决方案，因为与其他替代方案相比，它以较低的成本提供了来自两个全球知名合作伙伴 Esri 和 HERE 的各种现成地图样式。该服务还提供地理编码功能，可用于确定地址的标记位置。以下是如何组合这些功能以满足此用例的方法。

## 解决方案概述

对于此解决方案，我们将使用地图资源向在浏览器中运行的 MapLibre GL SDK ( [http://maplibre.org](http://maplibre.org/) )提供地图数据。我们将使用地点索引资源来执行从地址到位置的转换，并使用 Amazon Cognito 来管理将用于检索地图和地点数据的凭证。

![在用户浏览器中运行的 JavaScript 代码使用 Amazon Cognito 身份池 ID 获取凭证。 该代码使用凭证签署对 Amazon Location Service Map 资源的地图数据请求。 MapLibre GL SDK 使用生成的地图数据向用户显示交互式地图。 JavaScript 代码使用相同的凭据来签署对地点索引资源的地理编码请求，并使用生成的位置在地图上显示标记。](https://d2908q01vomqb2.cloudfront.net/0a57cb53ba59c46fc4b692527a38a87c78d84028/2021/07/12/architecture-diagram.png)

## 演练

实施此解决方案有三个主要步骤：

- 步骤 1：创建地图、地点索引和 Amazon Cognito 身份池资源
- 步骤 2：使用 JavaScript 将地图添加到 HTML 页面
- 第 3 步（可选）：使用地点对地址进行地理编码并在地图上显示标记

[![CloudFormation 启动堆栈按钮](https://d2908q01vomqb2.cloudfront.net/0a57cb53ba59c46fc4b692527a38a87c78d84028/2021/07/12/cf-launch-stack.png)](https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/quickcreate?templateUrl=https%3A%2F%2Famazon-location-blog-assets.s3.amazonaws.com%2Fadd-a-map-to-your-webpage%2Fresource-template.yaml&stackName=&param_Domain=&param_ResourceNamePrefix=)

### 先决条件

对于本演练，您应该具备以下先决条件：

- 一个[AWS 账户](https://signin.aws.amazon.com/signin?redirect_uri=https%3A%2F%2Fportal.aws.amazon.com%2Fbilling%2Fsignup%2Fresume&client_id=signup)
- 一个文本编辑器

### 步骤 1：创建地图、地点索引和 Cognito 身份池资源

首先，您必须创建地图、放置索引（如果您要使用地理编码来查找放置标记的位置的纬度和经度）、Cognito 身份池和关联的 AWS Identity and Access Management (IAM) 角色（因此您可以验证对地图和放置索引资源的请求）。对于此示例，我们将使用 Esri 导航矢量样式 (VectorEsriNavigation)，但您可以将其替换为此处引用的任何其他样式：[MapConfiguration – Amazon Location Service](https://docs.aws.amazon.com/location-maps/latest/APIReference/API_MapConfiguration.html). 您可以手动创建这些资源，但 CloudFormation 使这更方便。上述模板将创建所有必需的资源，并使用 IAM 角色策略中的 Condition 子句将生成的身份池 ID 的使用限制为指定域。<u>如果您在本地计算机上使用网络服务器尝试此操作，请使用`localhost:PORT`域，否则您将收到 HTTP 403 Forbidden 错误。</u>

#### 脚步：

1. 登录 CloudFormation 控制台
2. 选择**创建堆栈**>**使用新资源（标准）**
3. 添加 CloudFormation 模板的位置：[https](https://amazon-location-blog-assets.s3.amazonaws.com/add-a-map-to-your-webpage/resource-template.yaml) ://amazon-location-blog-assets.s3.amazonaws.com/add-a-map-to-your-webpage/resource-template.yaml
4. 单击**下一步**
5. 输入堆栈名称
6. 输入将托管页面的域名
7. 输入用于地图、地点索引和身份池资源名称的前缀
8. 单击**下一步**>**下一步**
9. 勾选**我承认 AWS CloudFormation 可能会创建 IAM 资源**的框。IAM 角色和相关策略将用于限制对地图和地点索引的访问。
10. 单击**创建堆栈**

AWS CloudFormation 的示例输出：

![CloudFormation 输出选项卡的屏幕截图。 该选项卡显示将在步骤 2 和 3 中使用的身份池 ID、地图名称和地点索引名称。](https://d2908q01vomqb2.cloudfront.net/0a57cb53ba59c46fc4b692527a38a87c78d84028/2021/07/12/cloudformation-outputs.png)

### 步骤 2：将地图添加到 HTML 页面

接下来，我们会将地图添加到您的网页。我们将使用 MapLibre SDK ( [https://maplibre.org](https://maplibre.org/) ) 以及来自 Amplify 和 Amazon Location 的一些帮助程序库。请记住用 CloudFormation 模板中的 IdentityPoolID 和 MapName 输出替换`YOUR_IDENTITY_POOL_ID`和`YOUR_MAP_NAME`。

这是代码：

```html
<!-- Copyright Amazon.com, Inc. or its affiliates. All Rights Reserved. -->
<!-- SPDX-License-Identifier: MIT-0 -->
<html>
  <head>
    <link
      href="https://unpkg.com/maplibre-gl@1/dist/maplibre-gl.css"
      rel="stylesheet"
    />
  </head>

  <body>
    <div id="map" style="height: 400px; width: 650px;"/>
    <script src="https://unpkg.com/maplibre-gl@1"></script>
    <script src="https://unpkg.com/amazon-location-helpers@1"></script>
	<script src="https://sdk.amazonaws.com/js/aws-sdk-2.937.0.min.js"></script>
    <script>
      async function initializeMap() {
        const map = await AmazonLocation.createMap(
          {
            identityPoolId: "YOUR_IDENTITY_POOL_ID",
          },
          {
            container: "map",
            center: [-123.1187, 49.2819], // initial map center point
            zoom: 10, // initial map zoom
            style: "YOUR_MAP_NAME",
            hash: true,
          }
        );

        map.addControl(new maplibregl.NavigationControl(), "top-left");
		
      }

      initializeMap();
    </script>
  </body>
</html>
```

HTML

### 第 3 步：向地图添加标记

最后，我们将在地图上添加一个标记。您可以使用以下代码轻松完成此操作`map.addControl()`。或者，如果您已经知道标记的位置，则可以跳过地理编码调用并仅使用`maplibregl.Marker().setLngLat([longitude,latitude]).addTo(map)`. 请记住用 CloudFormation 模板中的 IdentityPoolID 和 PlaceIndexName 输出替换`YOUR_IDENTITY_POOL_ID`和`YOUR_PLACE_INDEX_NAME`。

```js
// Find the location and put a marker on the map
const location = new AWS.Location({
  credentials: await AmazonLocation.getCredentialsForIdentityPool("YOUR_IDENTITY_POOL_ID"),
  region: "us-east-1"
});

const data = await location.searchPlaceIndexForText({
  IndexName: "YOUR_PLACE_INDEX_NAME",
  Text: "777 Pacific Blvd, Vancouver"
}).promise();

const position = data.Results[0].Place.Geometry.Point;
const marker = new maplibregl.Marker().setLngLat(position).addTo(map);
```

JavaScript

### 打扫干净

为避免将来产生费用，只需删除 CloudFormation 堆栈即可。转至**CloudFormation** > **Stacks**，选择您创建的堆栈并单击**Delete**。

## 结论

在这篇文章中，我们学习了如何快速轻松地向网页添加地图和相关标记。Amazon Location Service 使您能够以低成本执行此操作，并利用 Amazon Cognito 和 AWS Identity and Access Management (IAM) 的功能来控制对相关资源的访问。





------

**以下为备注：**

问题1：有一个403的错误，就是解决不了了，真奇怪

![image-20211011172134078](https://raw.githubusercontent.com/liangyimingcom/storage/master/PicGo/image-20211011172134078.png)

![image-20211011172141580](https://raw.githubusercontent.com/liangyimingcom/storage/master/PicGo/image-20211011172141580.png)



问题2：背景代码汇总：

~~~json
explore.map
https://aws-tc-largeobjects.s3.us-west-2.amazonaws.com/Curation/GettingStarted/LocationService/index.html.txt
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "MapsReadOnly",
      "Effect": "Allow",
      "Action": [
        "geo:GetMapStyleDescriptor",
        "geo:GetMapGlyphs",
        "geo:GetMapSprites",
        "geo:GetMapTile"
      ],
      "Resource": "arn:aws:geo:ap-northeast-1:xx:map/ExampleMap"
    }
  ]
}


Update CLI:
https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2-mac.html


aws location create-place-index --data-source Esri --index-name indexnametest01 --pricing-plan RequestBasedUsage  --region ap-northeast-1


aws location search-place-index-for-text --index-name indexnametest01 --text "beijing" --max-results 1
{
    "Results": [
        {
            "Place": {
                "Country": "CHN",
                "Geometry": {
                    "Point": [
                        116.39723000000004,
                        39.90750000000003
                    ]
                },
                "Label": "Beijing, CHN",
                "Municipality": "Beijing",
                "Region": "Beijing"
            }
        }
    ],
    "Summary": {
        "DataSource": "Esri",
        "MaxResults": 1,
        "ResultBBox": [
            116.39723000000004,
            39.90750000000003,
            116.39723000000004,
            39.90750000000003
        ],
        "Text": "beijing"
    }
}


aws location search-place-index-for-position --index-name indexnametest01 --position 116.39723000000004 39.90750000000003
{
    "Results": [
        {
            "Place": {
                "Country": "CHN",
                "Geometry": {
                    "Point": [
                        116.397428,
                        39.907478
                    ]
                },
                "Label": "北京市东城区西长安街",
                "Municipality": "北京市",
                "Neighborhood": "东城区",
                "Region": "北京市"
            }
        }
    ],
    "Summary": {
        "DataSource": "Esri",
        "MaxResults": 50,
        "Position": [
            116.39723000000004,
            39.90750000000003
        ]
    }
}

~~~

