<template>
  <div class="note-list">
    <head_bar></head_bar>
    <left_bar></left_bar>
    <el-col :span="16" :offset="6" class="grid-manage">
      <div class="grid-content bg-purple page-content">
        <article v-bind:id="'comment-' + item.weiboId" v-for="item in collectList" v-bind:key="item.weiboId">
          <div class="comment-meta">
            <div class="avatar">
              <router-link :to="{path:'/note',query: {id: item.userId}}">
                <img alt="" :src="URL_PREFIX + item.userHead" class="avatar-60 photo" height="60" width="60">
              </router-link>
            </div>
            <div class="caption">
              <h5 class="author">
                <span class="fn"><router-link :to="{path:'/note',query: {id: item.userId}}" rel="external nofollow" class="url">{{item.userName}}</router-link></span>
                    - <span class="comment-reply-link" href="#respond">发布</span>
              </h5>
              <p class="date">
                {{item.weiboTime}}
              </p>
            </div>
            <div class="block">
              <span class="wrapper">
                <el-button :plain="true" type="warning" @click="addFavorite(item)" :class="item.weiboisCollect?'btn-collected':''">{{item.weiboisCollect?'已收藏'+item.weiboCollect:'收藏'+item.weiboCollect}}</el-button>
                <el-button :plain="true" type="info" @click="openComments(item)">评论{{item.weiboComment}}</el-button>
              </span>
            </div>
          </div>
          <div v-if="item.rootWeibo==null && item.weiboPhoto != ''" class="comment-picture">
            <div class="media_box">
              <ul>
                <li>
                  <img :src="URL_PREFIX + item.weiboPhoto">
                </li>
              </ul>
            </div>
          </div>
          <div class="comment-body">
            <p v-if="item.weiboTopic"><router-link :to="'/topic/' + item.weiboTopic" class="theme-a">#{{item.weiboTopic}}# </router-link></p>
            <comment-part :originContent="item.weiboRootIds==-1||item.rootWeibo==null?item.weiboContent:item.weiboFwContent"></comment-part>
            <fw-content v-if="item.rootWeibo!=null" :data="item.rootWeibo!=null?item.rootWeibo:item"></fw-content>
            <p><router-link :to="{path:'/note',query: {username: at}}" v-if="at!=''" v-for="at in item.weiboAts" v-bind:key="at" class="at-a">@{{at}} </router-link></p>
          </div>
          <div class="comment-child" v-if="!item.open">
            <comment-post :pId="item.weiboId" :username="item.userName" @commentpost="addComment($event, item)"></comment-post>
            <div class="comment-child-meta">
              <div class="grid-content bg-purple" v-show="cItem.weiboId == item.weiboId" v-for="cItem in commentList" v-bind:key="cItem.date">
                <div class="avatar">
                  <router-link :to="{path:'/note',query: {id: cItem.userId}}">
                    <img alt="" :src="URL_PREFIX + cItem.userHead" class="avatar-20 photo" height="30" width="30">
                  </router-link>
                </div>
                <div class="caption">
                  <h5 class="author">
                    <span class="fn"><router-link :to="{path:'/note',query: {id: item.userId}}" rel="external nofollow" class="url">{{cItem.userName}}</router-link></span>
                    <span class="comment-reply-link" href="#respond">评论</span>
                  </h5>
                  <p class="date">
                    <a>{{cItem.commentTime}}</a>
                  </p>
                </div>
                <div class="comment-body">
                  <p>{{cItem.commentContent}}</p>
                </div>
              </div>
              <div class="load-comment" v-if="(commentList.length!=0 && commentList.length%3==0) || addCommentFlag==true">
                <a @click="loadmoreComments(item)" class="loadmore">点击查看更多评论</a>
              </div>
            </div>
          </div>
          </article>
          <el-row>
            <el-col :span="24" class="page-foot" v-if="(collectList.length!=0 && collectList.length%10==0) || delFlag == true">
              <a @click="loadmore" class="loadmore">点击查看更多</a>
            </el-col>
          </el-row>
      </div>
    </el-col>
    <foot_bar></foot_bar>
  </div>
</template>

<script>
import head_bar from './HeadBar.vue'
import foot_bar from './Foot.vue'
import left_bar from './LeftBar.vue'
import Comment from './Comment.vue'
import Fwcontent from './Fwcontent.vue'
import PostComment from './PostComment.vue'
import person from '../assets/person.png'

export default {
  // 我的收藏页面
  name: 'collect-list',
  created() {
    document.title = '我的收藏';
  },
  mounted() {
    // 初始化我的收藏列表
    if (this.isLogin=='false') {
      this.$router.push({path: '/'});
    }
    var _this = this;
    _this.page = 1;
    _this.$ajax({
      method: 'get',
      url: _this.URL_PREFIX + '/weibo/front/allmarks',
      params: {
        pageNum: _this.page,
        userId: localStorage.getItem('userId'),
        pageSize: 10
      }
    }).then(function(response) {
      if (response.data.code == 200) {
        _this.$message({
          showClose: true,
          message: response.data.description,
          type: 'success'
        });
        for (var i = 0; i < response.data.data.length; i++) {
          _this.collectList.push(response.data.data[i]);
        }
      }
      else {
        _this.$message({
          showClose: true,
          message: response.data.description,
          type: 'error'
        });
      }
    })
  },
  data () {
    return {
      addCommentFlag: false,
      page: 1,
      collectList: [],
      commentList: [],
      commentPage:[],
      commentAdd: [],
      collectAdd: 0,
      isLogin: localStorage.getItem('isLogin'),
      delFlag: false
    }
  },
  components: {
    head_bar,
    foot_bar,
    left_bar,
    'comment-part': Comment,
    'comment-post': PostComment,
    'fw-content': Fwcontent,
  },
  methods: {
    // 收藏取消收藏方法
    addFavorite: function(item) {
      var _this = this;
      if (item.weiboisCollect != true) {
        _this.$ajax({
          method: 'get',
          url: _this.URL_PREFIX + '/weibo/front/addmark',
          params: {
            userId: localStorage.getItem('userId'),
            weiboId: item.weiboId
          }
        }).then(function(response) {
          if (response.data.code == 200) {
            _this.$message({
              showClose: true,
              message: response.data.description,
              type: 'success'
            });
            item.weiboisCollect = true;
            item.weiboCollect++;
          }
          else {
            _this.$message({
              showClose: true,
              message: response.data.description,
              type: 'error'
            });
          }
        })
      }
      else {
        _this.$ajax({
          method: 'get',
          url: _this.URL_PREFIX + '/weibo/front/removemark',
          params: {
            userId: localStorage.getItem('userId'),
            weiboId: item.weiboId
          }
        }).then(function(response) {
          if (response.data.code == 200) {
            _this.$message({
              showClose: true,
              message: response.data.description,
              type: 'success'
            });
            item.weiboisCollect = false;
            item.weiboCollect--;
            _this.collectAdd--;
            _this.delFlag = true;
            for (var i = 0;i < _this.collectList.length;i++) {
              if (item.weiboId == _this.collectList[i].weiboId) {
                _this.collectList.splice(i, 1);
              }
            }
          }
          else {
            _this.$message({
              showClose: true,
              message: response.data.description,
              type: 'error'
            });
          }
        })
      }
    },
    // 加载更多评论
    loadmoreComments: function(item) {
      var _this = this;
      _this.$ajax({
        method: 'get',
        url: _this.URL_PREFIX + '/weibo/front/comment/getcomment',
        params: {
          page: 2,
          weiboId: item.weiboId,
          rows: (_this.commentPage[item.weiboId] - 1)*3 + _this.commentAdd[item.weiboId]
        }
      }).then(function(response) {
        if (response.data.code == 200) {
          var parsedata = JSON.parse(response.data.data);
          if (parsedata.rows.length < 3) {
            _this.addCommentFlag = false;
          }
          else {
            _this.addCommentFlag = true;
          }
          if (parsedata.rows.length > 3) {
            for (var i = 0; i < 3; i++) {
              _this.commentList.push(parsedata.rows[i]);
            }
            _this.commentPage[item.weiboId]++;
          }
          else {
            for (var i = 0; i < parsedata.rows.length; i++) {
              _this.commentList.push(parsedata.rows[i]);
            }
            _this.commentPage[item.weiboId]++;
          }
        }
        else {
          _this.$message({
            showClose: true,
            message: response.data.description,
            type: 'error'
          });
        }
      })
    },
    // 打开评论列表
    openComments: function(item) {
      if (item.open != false) {
        item.open=!item.open;
        var _this = this;
        if (!_this.commentPage[item.weiboId]) {
          _this.commentPage[item.weiboId] = 1;
          _this.commentAdd[item.weiboId] = 0;
          _this.loadmoreComments(item);
        }
      }
      else {
        item.open=!item.open;
      }
    },
    // 发表评论
    addComment: function(data, item) {
      this.commentList.unshift(data);
      this.addCommentFlag = true;
      this.commentAdd[item.weiboId]++;
      item.weiboComment++;
    },
    // 加载更多我的收藏
    loadmore: function() {
      var _this = this;
      _this.page++;
      _this.$ajax({
        method: 'get',
        url: _this.URL_PREFIX + '/weibo/front/allmarks',
        params: {
          pageNum: 2,
          userId: localStorage.getItem('userId'),
          pageSize: (_this.page-1)*10+_this.collectAdd
        }
      }).then(function(response) {
        if (response.data.code == 200) {
          _this.$message({
            showClose: true,
            message: response.data.description,
            type: 'success'
          });
          if (response.data.data.length > 10) {
            for (var i = 0; i < 10; i++) {
              _this.collectList.push(response.data.data[i]);
            }
          }
          else {
            for (var i = 0; i < response.data.data.length; i++) {
              _this.collectList.push(response.data.data[i]);
            }
          }
        }
        else {
          _this.$message({
            showClose: true,
            type: 'error',
            message: response.data.description
          })
        }
      })
    },
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.grid-manage {
  display: inline-block;
  position: absolute;
  height: 60%;
  width: 70%;
  margin-top: 2rem;
}
.comment-meta .avatar img {
  border-radius: 50%;
}
#search {
  display: inline-block;
  padding: 1rem 0 0.5rem 0;
  width: 100%;
}
.note-list {
  height: 100%;
  width: 100%;
}
.page-content {
	padding: 2rem 0 0 2rem;
  height: 100%;
  overflow-y: scroll;
}
.page-content::-webkit-scrollbar {
    width: 12px;
    background-color: #F5F5F5;
}
.page-content::-webkit-scrollbar-thumb {
    border-radius: 10px;
    -webkit-box-shadow: inset 0 0 2px rgba(0,0,0,.3);
    background-color: #324157;
}
.page-content::-webkit-scrollbar-track {
    -webkit-box-shadow: inset 0 0 2px rgba(0,0,0,0.3);
    border-radius: 10px;
    background-color: #F5F5F5;
}
.page-content article {
	padding: 1rem 0;
	border-top: 1px solid #ddd;
  width: 98%;
}
.page-content article .comment-meta {
  padding-left: 1rem;
	display: inline-block;
	width: calc(100% - 65px);
}
.page-content article {
	padding: 1rem 0;
	border-bottom: 1px solid #ddd;
}
.page-content article .comment-meta {
  padding-left: 1rem;
	display: inline-block;
	width: calc(100% - 65px);
}
.page-content article .comment-meta .avatar,.caption {
  display: inline-block;
}
.page-content article .comment-body {
  padding: 0.5rem 1rem;
  font-size: 14px;
  word-break:break-all;
  letter-spacing: 1.5px;
}

.copyright {
	text-align: center;
	color: #fff;
}
.block {
  float: right;
}
.page-foot {
  text-align: center;
  margin: 1rem 0;
}
.loadmore {
  cursor: pointer;
  text-decoration: underline;
}
.at-a,.theme-a {
  cursor: pointer;
}
.theme-a {
  color: #eb7350;
}
.loadmore:hover {
  text-decoration: none;
}
.read-comment-child {
  float: right;
}
.comment-child-meta .caption,.avatar {
  display: inline-block;
}
.comment-child-meta {
  padding: 0 1rem;
}
.comment-child-meta .grid-content {
  padding: 0 5px;
}
.comment-child-meta .grid-content p {
  font-size: 12px;
}
.comment-child-meta .comment-body {
  display: inline-block;
  max-width: 75%;
  word-break:break-all;
  letter-spacing: 1.5px;
}
.comment-child-meta .caption h5,a {
  margin: 0;
}

.load-comment {
  margin-top: 0.2rem;
  text-align: center;
  font-size: 12px;
}
.type-section {
  background: #f3f3f3;
  padding: 1rem 1rem;
  text-align: center;
}
.type-section span {
  font-size: 1rem;
  text-align: center;
}
.date {
  color: #999999;
  font-size: 14px;
}
.comment-picture {
  display: inline-block;
  width: calc(100% - 65px);
}
.comment-picture .media_box {
  width:50%;
  min-width: 350px;
}
.comment-picture ul li {
  width: 300px;
  height: 200px;
  margin: 4px 0 0 4px;
  list-style: none;
  float: left;
  overflow: hidden;
}
.comment-picture ul li img {
  width: 300px;
  height: auto;
  display: inline-block;
  vertical-align: top;
}
#facebox ul li {
  display: inline-block;
  list-style: none;
}
.btn-collected {
  color:  #dea726;
  border-color: #dea726;
}
</style>
