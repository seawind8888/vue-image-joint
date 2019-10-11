# vue-image-joint
基于vue的图片拼接组件，可返回base64用于上传

## Preview
![image](/screenshot/preview.gif)

## Install

```bash
npm i vue-image-joint
```

## Usage

Plugin install:

```js
import Vue from 'vue'
import ImageJoint from 'vue-image-joint'

Vue.use(ImageJoint)
```

vue file:

```JavaScript
    <ImageJoint 
        ref="joint"
        :width='800'
    />
    ...
    methods: {
        getImage($event) {
            this.imgSrc = this.$refs["joint"].getImageDataURL(); // get image base64Url
        },
    },

```

## Props


| name            | type                             | default    | description                                                            |
| --------------- | -------------------------------- | ---------- | ---------------------------------------------------------------------- |
| width           | Number                           | 500       | 宽度（导出图片为此宽） 
| info            | String                           | "点击添加或拖拽此处"       | 图片选择提示信息
| encoder         | Number                           | 0.5       | 图片质量

## Events

| name     | description                                              | $event                            
| -------- | -------------------------------------------------------- | ---------------------------------- |
| getImageDataURL   |    获取图片base64                         | base64Url      |


## License

MIT
