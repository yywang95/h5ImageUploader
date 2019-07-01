<template>
    <div class="doing-component">
        <div class="doing-content">
            <img class="doing-icon" src="//img.58cdn.com.cn/lbg/jianzhan/images/loading.gif" />
            <div class="doing-text">{{text}}</div>
        </div>
    </div>
</template>
<script>
/**
 * @description loading组件，类似微信的wx.showLoading
 */
export default {
    props: {
        mask: {
            type: Boolean,
            default: true,
        },
        text: {
            type: String,
            default: '上传中',
        },
    },
};
</script>
<style lang="scss" scoped>
.doing-component {
    z-index: 999;
    position: fixed;
    top: 0;
    bottom: 0;
    left: 0;
    right: 0;
}
.doing-content {
    position: absolute;
    top: 0;
    bottom: 0;
    left: 0;
    right: 0;
    margin: auto;
    padding: 40px 0;
    width: 200px;
    height: 100px;
    text-align: center;
    color: #fff;
    border-radius: 5px;
    background: rgba(0, 0, 0, .6);
}
.doing-text {
    font-size: 28px;
}
.doing-icon {
    display: inline-block;
    margin-bottom: 25px;
    width: 40px;
}
</style>
