<template>
    <div class="ratings" ref="ratings">
        <div class="ratings-content">
            <div class="overview">
                <div class="overview-left">
                    <h1 class="score">{{seller.score}}</h1>
                    <div class="title">综合评分</div>
                    <div class="rank">高于周边商家{{seller.rankRate}}%</div>
                </div>
                <div class="overview-right">
                    <div class="score-wrapper">
                        <span class="title">服务态度</span>
                        <star :size='36' :score='seller.serviceScore'></star>
                        <span class="score">{{seller.serviceScore}}</span>
                    </div>
                    <div class="score-wrapper">
                        <span class="title">商品评分</span>
                        <star :size='36' :score='seller.foodScore'></star>
                        <span class="score">{{seller.foodScore}}</span>
                    </div>
                    <div class="deliver-wrapper">
                        <span class="title">送达时间</span>
                        <span class="sendTime">{{seller.deliveryTime}}分钟</span>
                    </div>
                </div>
            </div>
            <split></split>
            <ratingselect :selectType='selectType' :onlyContent='onlyContent' :ratings='ratings'></ratingselect>
            <div class="rating-wrapper">
                <ul>
                    <li v-show="needShow(rating.rateType,rating.text)" v-for="rating in ratings" class="rating-item border-1px" :key="rating">
                        <div class="avater">
                            <img :src="rating.avatar" width="28" height="28">
                        </div>
                        <div class="content">
                            <h1 class="name">{{rating.username}}</h1>
                            <div class="star-wrapper">
                                <star :size='24' :score='rating.score'></star>
                                <span class="deliver" v-show='rating.deliveryTime'>
                                    {{rating.deliveryTime}}
                                </span>
                            </div>
                            <p class="text">{{rating.text}}</p>
                            <div class="recommend" v-show='rating.recommend && rating.recommend.length'>
                                <span class="icon-thumb_up"></span>
                                <span class="item" v-for="item in rating.recommend" :key="item">{{item}}</span>
                            </div>
                            <div class="time">
                                {{rating.rateTime | formatDate}}
                            </div>
                        </div>
                    </li>
                </ul>
            </div>
        </div>
    </div>
</template>

<script type="text/ecmascript-6">
    // 定义满意与不满意、全部分组常量
    const POSITIVE = 0;
    const NEGATIVE = 1;
    const All = 2;
    const ERR_OK = 0;
    import star from '../star/star';
    import split from '../split/split';
    import BScroll from 'better-scroll';
    import ratingselect from '../ratingselect/ratingselect';
    import {formatDate} from '../../common/js/date';
    import {baseUrl} from '../../api/config.js';
    import {getAppData} from 'api/appData';
    export default {
        props: {
            seller: {
                type: Object
            }
        },
        data() {
            return {
                ratings: [],
                selectType: All,
                onlyContent: false
            };
        },
        methods: {
            // 通过判断该条评价的类型和是否有评价内容来确认该条评价是否显示
            needShow(type, text) {
            if (this.onlyContent && !text) {
                return false;
            }
            if (this.selectType === All) {
                return true;
            } else {
                return type === this.selectType;
            }
            }
        },
        created() {
            // let _that = this;
            // this.$axios.get(baseUrl).then(function(response) {
            //     _that.ratings = response.data.ratings;
            //     _that.$nextTick(() => {
            //     _that.scroll = new BScroll(_that.$refs.ratings, {
            //             click: true
            //         });
            //     });
            //   })
            //   .catch(function(err) {
            //     console.log(err);
            // });
            // getAppData().then((res) => {
            //     console.log(res);
            //     this.ratings = res.data.ratings;
            //     this.$nextTick(() => {
            //     this.scroll = new BScroll(this.$refs.ratings, {
            //             click: true
            //         });
            //     });
            // });
            this.$http.get('/api/ratings').then((response) => {
                response = response.body;
                if (response.errno === ERR_OK) {
                    this.ratings = response.data;
                    this.$nextTick(() => {
                        this.scroll = new BScroll(this.$refs.ratings, {
                            click: true
                        });
                    });
                }
            });
            // 接收子组件派发的事件，并定义该事件（2.0）
            this.$root.eventHub.$on('ratingType.select', (type) => {
                this.selectType = type;
                this.$nextTick(() => {
                    this.scroll.refresh();
                });
            });
            this.$root.eventHub.$on('content.toggle', (onlyContent) => {
                this.onlyContent = onlyContent;
                this.$nextTick(() => {
                    this.scroll.refresh();
                });
            });
        },
        //  时间转换
        filters: {
            formatDate(time) {
            let data = new Date(time);
            return formatDate(data, 'yyyy-MM-dd HH:mm');
            }
        },
        components: {
            star, split, ratingselect
        }
    };
</script>

<style lang='stylus' rel='stylesheet/stylus'>
    @import '../../common/stylus/mixin'
    .ratings
        position :absolute;
        top :174px
        bottom :0
        left :0
        width :100%
        overflow :hidden
        .overview
            display :flex
            padding :18px 0
            .overview-left
                flex :0 0 137px
                padding:6px 0
                width :137px
                border-right :1px solid rgba(7,17,27,.1)
                text-align :center
                @media only screen and (max-width: 320px)
                    flex :0 0 120px
                    width :120px
                .score
                    margin-bottom :6px
                    line-height :28px
                    font-size :24px
                    color :rgb(255,153,0)
                .title
                    margin-bottom :8px
                    line-height :12px
                    font-size :12px
                    color :rgb(7,17,27)
                .rank
                    font-size :10px
                    line-height :10px
                    color :rgb(147,153,159)
            .overview-right
                flex :1
                padding :6px 0 6px 24px
                @media only screen and (max-width: 320px)
                    padding:6px 0 6px 12px
                .score-wrapper
                    margin-bottom :8px
                    font-size :0
                    .title
                        display :inline-block;
                        vertical-align :top
                        font-size :12px
                        line-height :18px
                        color :rgb(7,17,27)
                    .star
                        display :inline-block;
                        vertical-align :top
                        margin :0 12px
                        .star-item
                            margin-right:4px
                    .score
                        display :inline-block;
                        vertical-align :top
                        font-size :12px
                        line-height :18px
                        color :rgb(255,253,0)
                .deliver-wrapper
                    font-size :0
                    .title
                        display :inline-block;
                        vertical-align :top
                        font-size :12px
                        line-height :18px
                        color :rgb(7,17,27)
                        margin-right :12px
                    .sendTime
                        font-size :12px
                        line-height :18px
                        color :rgb(147,153,159)
        .rating-wrapper
            padding :0 18px
            .rating-item
                display :flex
                padding :18px 0
                border-1px(rgb(7,17,27))
                .avater
                    flex :0 0 28px
                    width :28px
                    margin-right:12px
                    img
                        border-radius :50%
                .content
                    position :relative
                    flex :1
                    .name
                        line-height :12px
                        font-size :10px
                        color :rgb(7,17,27)
                        margin-bottom :4px
                    .star-wrapper
                        margin-bottom :6px
                        font-size :0
                        .star
                            display :inline-block
                            vertical-align :top
                            margin-right :16px
                            .deliver
                                display :inline-block
                                line-height :12px
                                font-size :10px
                                color :rgb(147,153,159)
                    .text
                        margin-bottom :8px
                        line-height :18px
                        color :rgb(7,17,27)
                        font-size :12px
                    .recommend
                        line-height :16px
                        font-size :0
                        .icon-thumb_up,.item
                            display :inline-block
                            margin :0 8px 4px 0
                            font-size :9px
                        .icon-thumb_up
                            color :rgb(0,160,220)
                        .item
                            padding :0 6px
                            border:1px solid rgba(7,17,27,.1)
                            border-radius :1px
                            color :rgb(147,153,159)
                            background :#fff
                    .time
                        position :absolute
                        right :0
                        top :0
                        line-height :12px
                        font-size :10px
                        color :rgb(147,153,159)
</style>
