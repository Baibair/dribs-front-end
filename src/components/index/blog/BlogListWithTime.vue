<template>
    <div id="blog-list-with-time" class="clearfix">
        <el-col :span="24">
            <el-timeline >
                <el-timeline-item :timestamp="getTime(item.postTime)" v-for="(item,index) in blogList" :key="item.blogId" placement="top">
                    <BlogBriefInfo

                                    :blog="item"
                                   :is-profile="isProfile"
                                   @toggleLike="toggleLike(index)"
                                   @onDelete="deleteBlog(item.blogId,index)"
                    />
                    <!--:blog-id="item.blogId" :blog-content="item.blogContent" :repost-count="item.repostCount"-->
                    <!--:img-list="item.imgs" :is-repost="item.repost" :repost-path="item.repostPath"-->
                    <!--:source="item.source" -->
                    <!--:reply-count="item.replyCount" :like-count="item.likeCount" :user="item.author"-->
                    <!--:isLiked = "item.isLiked"-->
                </el-timeline-item>
            </el-timeline>
        </el-col>

    </div>
</template>

<script>
    import BlogBriefInfo from "./BlogBriefInfo"
    import {request} from "../../../network/request"
    export default {
        name: "BlogListWithTime",
        props: ['blogList','isProfile'],
        data(){
          return{

          }
        },
        methods: {
            getTime(timestamp){
                let date = new Date(timestamp);
                let now = new Date();
                let nowTime = now.getTime();
                let blogTime = date.getTime();
                let def = nowTime-blogTime;
                if(now.getDate() === date.getDate()){
                    if(def<60000){
                        return "刚刚"
                    } else if(def<3600000){
                        return Math.floor(def/60000)+"分钟前";
                    } else if(def<24*3600000){
                        return Math.floor(def/3600000)+"小时前";
                    }
                }

                let YY = date.getFullYear() + '年';
                let MM = (date.getMonth() + 1 < 10 ? '0' + (date.getMonth() + 1) : date.getMonth() + 1) + '月';
                let DD = (date.getDate() < 10 ? '0' + (date.getDate()) : date.getDate())+"日";
                let hh = (date.getHours() < 10 ? '0' + date.getHours() : date.getHours()) + ':';
                let mm = (date.getMinutes() < 10 ? '0' + date.getMinutes() : date.getMinutes()) ;
                // let ss = (date.getSeconds() < 10 ? '0' + date.getSeconds() : date.getSeconds());
                return YY + MM + DD +" "+hh + mm;
            },
            toggleLike(index){
                this.checkAndAction( () => {
                    let url = this.blogList[index].isLiked? 'blog/cancelLikeBlog':'blog/likeBlog';
                    request({
                        url: url,
                        params: {
                            blogId: this.blogList[index].blogId
                        }
                    }).then( res => {
                        if(res.data){
                            if(this.blogList[index].isLiked){
                                this.blogList[index].likeCount--;
                                this.blogList[index].isLiked = false;
                            }else {
                                this.blogList[index].likeCount++;
                                this.blogList[index].isLiked = true;
                            }
                        }else {
                            this.$alert("操作失败");
                        }
                    }).catch( err => {
                        this.$alert("系统错误");
                    })
                },this.$route.fullPath)
            },
            deleteBlog(blogId,index){
                request({
                    url: 'blog/deleteBlog',
                    method: 'POST',
                    data: {
                        blogId
                    }
                }).then( res => {
                    if(res.statusCode === '000000'){
                        this.$message.success('删除成功');
                    } else {
                        this.$message.error(res.message);
                    }
                }).catch( err => {
                    this.$message.error('系统错误');
                });
                this.blogList.splice(index,1);
                // this.$alert("123456");
            }
        },
        components: {
            BlogBriefInfo
        }
    }
</script>

<style scoped>

</style>                   