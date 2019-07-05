<template>
    <div class="images-component">
        <div
            :key="item+index"
            v-for="(item, index) in items"
            :style="{backgroundImage: `url(${item}?w=200&h=200)`}"
            class="images-item images-preview">
            <i v-if="download" class="images-down" title="下载图片" @click="downloadImg(item)"></i>
            <i v-if="!disabled" class="images-del" @click="handleDel(item, index)"></i>
        </div>
        <div v-if="items.length < limit && ((disabled && items.length < 1) || !disabled)" class="images-item images-add">
            <Uploader
                accept="image/png, image/bmp, image/jpeg, image/jpg, image/gif"
                :multiple="multiple"
                :disabled="disabled"
                :limit="limit"
                :limitsize="limitsize"
                @on-error="listenUploadError"
                @on-success="listenUploadSuccess">
                <div class="images-up">
                    <i class="images-up-icon"></i>
                    <div class="images-up-text">点击添加图片</div>
                </div>
            </Uploader>
        </div>
        <div class="images-number">{{items.length}}/{{limit}}</div>
    </div>
</template>
<script>
/**
 * @description 图片上传组件
 */
import Uploader from '.Uploader';
import { dataURLtoFile } from '@/utils/imageUtil';

export default {
    name: 'imagesPC',
    components: { Uploader },
    props: {
        id: {
            type: [String, Number], // 区别多个组件
            default: '',
        },
        limit: {
            type: Number, // 最多上传几张图片
            default: 1,
        },
        limitsize: {
            type: Number, // 图片限制大小
            default: 0,
        },
        items: {
            type: Array, // 初始显示图片的src Array[String]
            default() {
                return [];
            },
        },
        multiple: {
            type: Boolean, // 是否可多选
            default: false,
        },
        disabled: {
            type: Boolean, // 是否可修改
            default: false,
        },
        download: {
            type: Boolean, // 是否可以下载图片
            default: false,
        },
    },
    methods: {
        // 文件上传错误
        listenUploadError(...args) {
            this.$emit('on-error', this.id, ...args);
        },
        // 文件上传成功
        listenUploadSuccess(urls) {
            this.$emit('on-success', this.id, urls);
        },
        // 删除图片
        handleDel(item, index) {
            this.$emit('on-delete', this.id, item, index);
        },
        // 图片下载
        downloadImg(url) {
            const imgName = url.split('/').slice(-1)[0] || '';
            this.downloadIamge(url, imgName);
        },
        // 下载图片地址和图片名
        downloadIamge(imgsrc, name) {
            try {
                const image = new Image();
                // 解决跨域 Canvas 污染问题
                image.setAttribute('crossOrigin', 'anonymous');
                image.onload = () => {
                    // 将图片绘制在canvas上
                    const canvas = document.createElement('canvas');
                    canvas.width = image.width;
                    canvas.height = image.height;
                    const context = canvas.getContext('2d');
                    context.drawImage(image, 0, 0, image.width, image.height);

                    // 将canvas转为base64, 得到图片的base64编码数据
                    const url = canvas.toDataURL('image/png');

                    // 图片太大，base64会很大，导致a.href放不下，下载网络错误，所以将base64转为本地文件
                    const imageFile = dataURLtoFile(url, name);
                    const imageBlob = URL.createObjectURL(imageFile);

                    // 生成一个a元素, 创建一个单击事件
                    const a = document.createElement('a');
                    const event = new MouseEvent('click');

                    a.download = name; // 设置图片名称
                    a.href = imageBlob; // 将生成的URL设置为a.href属性
                    a.dispatchEvent(event); // 触发a的单击事件
                };
                image.src = imgsrc;
            } catch (e) {
                console.error(`下载图片失败：${e.toString()}`);
                window.open(imgsrc);
            }
        },
    },
};
</script>
<style lang="scss" scoped>
@import '../../assets/styles/const-pc';
.images-component {
    overflow: hidden;
}
.images-item {
    float: left;
    width: 120px;
    height: 120px;
    border: 1px solid #E7E7EB;
    border-radius: 2px;
    background-color: #f6f6f6;
    & + .images-item {
        margin-left: 20px;
    }
    &:hover {
        .images-down {
            display: block;
        }
    }
}
.images-preview {
    position: relative;
    background-size: cover;
    background-position: center;
}
.images-del {
    position: absolute;
    top: 5px;
    right: 5px;
    width: 18px;
    height: 18px;
    /* 你自己项目的删除图标按钮 */
    background-image: url('icon-delete.png');
    background-size: cover;
    background-position: center;
    cursor: pointer;
}
.images-down {
    display: none;
    position: absolute;
    top: 40px;
    left: 40px;
    width: 40px;
    height: 40px;
    /* 你自己项目的下载图标按钮 */
    background-image: url('icon-download.png');
    background-size: cover;
    background-position: center;
    cursor: pointer;
}
.images-add {
    font-size: 40px;
    color: #9b9b9b;
    line-height: 100px;
    text-align: center;
}
.images-up {
    position: relative;
    width: 120px;
    height: 120px;
    cursor: pointer;
    .images-up-icon {
        position: absolute;
        top: 0;
        bottom: 20px;
        left: 0;
        right: 0;
        margin: auto;
        width: 40px;
        height: 40px;
        /* 你自己项目的添加图标按钮 */
        background-image: url('icon-add.png');
        background-size: cover;
        background-position: center;
    }
    .images-up-text {
        padding-top: 90px;
        font-size: 14px;
        line-height: 20px;
        color: #333;
    }
}
.images-number {
    float: left;
    position: relative;
    top: 105px;
    margin-left: 20px;
    font-size: 12px;
    color: #333;
}
</style>
