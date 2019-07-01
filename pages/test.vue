<template>
    <div>
        <Images
            :id="'pics'"
            :items="pics"
            @on-error="listenUploadError"
            @on-success="listenUploadSuccess"
            @on-delete="listenUploadDelete" />
    </div>
</template>
<script>
import Images from '@/components/Images';

export default {
    data() {
        return {
            pics: [],
        };
    },
    methods: {
        /** @desc 图片上传错误 */
        listenUploadError(id, msg, item) {
            console.error(`图片${item.name}上传失败：${msg}`);
        },
        /** @desc 图片上传成功 */
        listenUploadSuccess(id, urls) {
            this[id].push(...urls);
        },
        /** @desc 图片删除 */
        listenUploadDelete(id, item, index) {
            this[id].splice(index, 1);
        },
    },
};
</script>