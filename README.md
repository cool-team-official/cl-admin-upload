# upload 文件上传

## Options

组件配置参数

```js
{
    // 上传的地址
	action: "",
	// 上传的文件类型
	accept: "",
	// 上传的文件字段名
	name: "file",
	// 尺寸
	size: "128px",
	// 显示图标
	icon: "el-icon-upload",
	// 显示文案
	text: "选择文件"
}
```

## Service

| 服务名称   | 说明     |
| ---------- | -------- |
| space.info | 文件信息 |
| space.type | 文件类型 |

## Components

### cl-upload

文件上传

#### 参数

| 参数            | 说明                                                          | 类型                    | 可选值 | 默认值         |
| --------------- | ------------------------------------------------------------- | ----------------------- | ------ | -------------- |
| v-model / value | 绑定值                                                        | string / array          |        |                |
| action          | 上传的地址                                                    | string                  |        |                |
| size            | 图片大小                                                      | array / string / number |        | 128px          |
| icon            | 图标                                                          | string                  |        | el-icon-upload |
| text            | 文本                                                          | string                  |        | 选择文件       |
| accept          | 接受上传的文件类型                                            | string                  |        |                |
| multiple        | 是否支持多选文件                                              | boolean                 |        |                |
| limit           | 最大允许上传个数                                              | number                  |        |                |
| more...         | [其他参数](https://element.eleme.cn/#/zh-CN/component/upload) |                         |        |                |

### cl-upload-space

文件空间

#### 参数

| 参数       | 说明                            | 类型    | 可选值 | 默认值 |
| ---------- | ------------------------------- | ------- | ------ | ------ |
| action     | 上传的地址，同 cl-upload.action | string  |        |        |
| limit      | 选择图片的长度                  | number  |        | 8      |
| disabled   | 是否禁用                        | boolean |        |        |
| headers    | 设置上传的请求头部              | object  |        |        |
| data       | 上传时附带的额外参数            | object  |        |        |
| accept     | 上传的文件类型                  | string  |        |        |
| detailData | 是否返回详细数据                | boolean |        |        |
| showButton | 是否显示按钮                    | boolean |        | true   |

## 示例

默认 oss 签名上传

```html
<cl-upload></cl-upload>
```

本地接口上传 /comm/upload

```html
<cl-upload action="comm"></cl-upload>
```

指定类型上传 accept=.jpg,.png

```html
<cl-upload accept=".jpg,.png"></cl-upload>
```

多图上传 picture-card

```html
<cl-upload multiple :limit="3" listType="picture-card"></cl-upload>
```

文件上传 file

```html
<cl-upload accept="*" list-type="file"></cl-upload>
```

自定义

```html
<cl-upload icon="el-icon-picture" text="选择图片" :size="[120, 200]"></cl-upload>
```

拖拽上传

```html
<cl-upload drag>
	<i class="el-icon-upload"></i>
	<div class="el-upload__text">将文件拖到此处，或<em>点击上传</em></div>
</cl-upload>
```

文件空间

```html
<cl-upload-space></cl-upload-space>
```
