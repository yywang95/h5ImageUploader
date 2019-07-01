<template>
    <div class="uploader-component" @click="handleClick">
        <input
            ref="input"
            type="file"
            :multiple="multiple"
            :accept="accept"
            :disabled="disabled"
            @change="handleChange">
        <slot></slot>
        <Doing v-if="submiting && loading" />
    </div>
</template>
<script>
/**
 * @description 上传组件
 */
import Doing from './Doing';
import { dataURLtoFile, rotateImg, getOrientation, compressImg } from '@/utils/imageUtil';

export default {
    name: 'uploaderPC',
    components: { Doing },
    props: {
        multiple: {
            type: Boolean, // 是否可多选
            default: false,
        },
        limit: {
            type: Number,
            default: 1, // 限制文件数量
        },
        limitsize: {
            type: Number, // 限制文件大小(字节 1M = 1024kb = 1024*1024b字节)，设为0表示不限制
            default: 0,
        },
        limitIntercept: {
            type: Boolean, // 如果有文件超出了大小限制，是否对此次上传做拦截，直接返回onError，当选择非图片类型的时候有用，上传图片类型建议传false，因为我们会对图片进行压缩
            default: false,
        },
        accept: {
            type: String, // 可接受资源类型
            default: '*',
        },
        action: {
            type: String, // 上传服务器地址
            default: '',
        },
        disabled: {
            type: Boolean, // 是否不可用
            default: false,
        },
        loading: {
            type: Boolean, // 是否显示loading图标
            default: false,
        },
    },
    data() {
        return {
            submiting: false, // 是否在上传中
        };
    },
    computed: {
        actionUrl() {
            return this.action || '/fileUpload';
        },
    },
    methods: {
        /** @desc 选择文件 */
        handleClick() {
            if (this.submiting) return;
            this.$refs.input.click();
        },
        /** @desc 选择文件改变 */
        handleChange(e) {
            const { files } = e.target;

            // 未选择文件的时候不做处理
            if (!files) return;

            // 上传文件
            this.uploadFiles(files);

            // 重置file表单
            this.$refs.input.value = null;
        },
        /** @desc 上传文件 */
        async uploadFiles(files) {
            let postFiles = Array.prototype.slice.call(files);

            // 只取限制的文件数量
            const limit = this.multiple ? this.limit : 1;
            postFiles = postFiles.slice(0, limit);

            // 如果设置了对超过大小的文件进行拦截，则判断文件大小是否在限制内
            if (this.limitIntercept) {
                for (let i = 0, len = postFiles.length; i < len; i += 1) {
                    if (this.limitsize > 0 && postFiles[i].size > this.limitsize) {
                        this.$emit('on-error', '大小超出限制', postFiles[i]);
                        return;
                    }
                }
            }

            // 开始上传
            this.submiting = true;

            let urlList = await this.upload(postFiles);
            urlList = urlList.filter(item => item.length);

            // 结束上传
            this.submiting = false;

            // url的列表数量和选择文件的数量不一样则表示上传的时候有文件上传错误，则此次上传无效
            if (urlList.length !== postFiles.length) return;

            this.$emit('on-success', urlList);
        },
        /** @desc 批量提交上传的文件 */
        async upload(files) {
            const self = this;
            const urlList = await Promise.all(files.map(async (file) => {
                const formData = new FormData();

                // 如果当前文件是图片类型，则对图片做一些处理，返回处理后的图片文件流
                if (/^image/.test(file.type)) {
                    // 对移动端拍照的时候旋转做处理
                    file = await self.imageHandle(file);
                }

                formData.append('source', file);
                try {
                    // 提交上传请求
                    // 请在这里写你自己项目的请求方法
                    const data = await self.$request.upload(self.actionUrl, formData);
                    return data.data && data.data.source;
                } catch (e) {
                    self.$emit('on-error', e.toString(), file);
                    return '';
                }
            }));
            return urlList;
        },
        /** @desc 图片处理 */
        async imageHandle(file) {
            const self = this;

            // 获取图片旋转信息，后面方法会用
            const or = await getOrientation(file, 'Orientation');

            return new Promise((resolve, reject) => {
                // 图片的处理用到了下面这些方法，如果当前系统不支持，就直接返回源文件
                if (!window.FileReader || !window.atob) return resolve(file);

                // 使用FileReader读取文件流
                const reader = new FileReader();
                reader.readAsDataURL(file);
                /* eslint-disable func-names */
                // 箭头函数会改变this，所以这里不能用肩头函数
                reader.onloadend = function () {
                    const result = this.result;
                    const img = new Image();
                    img.src = result;
                    img.onload = function () {
                        const data = self.imageHandleStep(img, or);
                        const f = dataURLtoFile(data);
                        resolve(f);
                    };
                };
            });
        },
        /** @desc 开始对图片文件进行处理 */
        imageHandleStep(img, or) {
            // 图片的所有处理都是基于canvas的，所以先将图片转为canvas
            const canvas = document.createElement('canvas');
            const ctx = canvas.getContext('2d');

            // 图片原始大小
            const width = img.width;
            const height = img.height;
            canvas.width = width;
            canvas.height = height;
            ctx.drawImage(img, 0, 0, width, height);

            // step1. 如果设置了图片的大小限制，则对超过大小的图片进行压缩
            const imgSize = img.src.length;
            if (this.limitsize > 0 && imgSize > this.limitsize) {
                compressImg(img, canvas, this.limitsize);
            }

            // step2. 用手机拍照的时候，iphone手机拿的方向不同上传的图片会被旋转，所以需要对旋转的图片转回到正确的方向
            // 不在异常范围内的文件不做旋转处理
            if ([6, 8, 3].indexOf(~~or) > -1) {
                this.getRotateImg(img, canvas, or);
            }

            return canvas.toDataURL('image/png', 1);
        },
        /**
         * @desc 获取旋转后的图片
         * @param {Object} img 图片文件
         * @param {Object} canvas 画布
         * @param {Number} or 旋转信息
         * @param {Object} compressSize 指定宽高
         */
        getRotateImg(img, canvas, or) {
            switch (or) {
            case 6: // 顺时针旋转90度
                rotateImg(img, 'right', canvas);
                break;
            case 8: // 逆时针旋转90度
                rotateImg(img, 'left', canvas);
                break;
            case 3: // 顺时针旋转180度
                rotateImg(img, 'right', canvas, 2);
                break;
            default:
                break;
            }
        },
    },
};
</script>
<style lang="scss" scoped>
.uploader-component {
    position: relative;
}
input[type="file"] {
    display: none;
}
</style>
