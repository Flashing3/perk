<template>
	<div id="PerkArticle">
		<div class="container">
			<h2 class="article-title">{{ article.title }}</h2>
			<div class="row">
				<span class="article-time text-primary">{{ article.pub_date.toLocaleDateString() }}</span>&nbsp;&nbsp;
				<span class="article-author">{{ article.authors }}</span>
			</div>
			<div class="article-img">
				<img :src="article.img" />
			</div>
			<div v-if="articleFolded" class="article-content">
				{{{ article.brief }}}
			</div>
			<div v-else="articleUnfoled" class="article-content">
				{{{ article.content }}}
			</div>
			<p class="article-toggle">
				<a href="#" @click.prevent="articleToggle">{{ articleOp }}</a>
			</p>
		</div>
		<div class="row">
			<perk-comment :hot-comments="cHot" :all-comments="cAll" :me-comments="cMe"></perk-comment>
		</div>
	</div>
</template>

<script>
	export default{
		data: function(){
			return {
				articleFolded: true,
				articleOp: '展开全文 >>',
				article: {},
				cAll: [],
				cMe: []
			}
		},
		computed: {
			cHot: function(){
				var _cHot = Object.assign([], this.cAll);
				_cHot.sort(function(a, b){
					var ahot = a.n_replies*3 + a.n_likes,
						bhot = b.n_replies*3 + a.n_likes;
					if(ahot > bhot)
						return -1;
					else
						return 1;
				});
				return _cHot;
			}
		},
		watch: {
			articleFolded: function(val){
				if(val)
					this.articleOp = '展开全文 >>';
				else
					this.articleOp = '收起全文 <<';
			}
		},
		methods: {
			articleToggle: function(event){
				this.articleFolded =! this.articleFolded;
			},
			getMeComments: function(){
				var _cMe = [],
					comments = this.cAll;
				if(window.$.cookie('user'))
					for(var key in comments){
						var userid = parseInt(window.$.cookie('user'));
						if(userid == comments[key].user.id){
							_cMe.push(comments[key]);
						}else{
							for(var rs in comments.replys){								
								if(userid == comments.replys[rs].user.id){
									_cMe.push(comments[key]);
									break;
								}
							}
						}
					}
				window._me_comments = _cMe;
				return _cMe;
			},
			updateMeComments: function(){
				var _cMe = this.getMeComments();
				this.$set('cMe', _cMe);
			}
		},
		events: {
			CallUpdateMeComments: function(){
				this.updateMeComments();
			}
		},
		ready: function(){
			var urlstr = window._server_url + 'article/' + this.$route.params.page,
				self = this,
				article_obj = {};
			window.$.ajax({
				dataType: 'json',
				url: urlstr,
				success: function(res){
					article_obj.title = res.title;
					article_obj.brief = res.brief;
					article_obj.img = res.img;
					article_obj.content = res.content;
					article_obj.pub_date = new Date(res.pub_date);
					var users = res.users,
						authors = [];
					for(var key in users)
						authors.push(users[key].profile.nick_name);
					article_obj.authors = authors.join(' ');
					self.$set('article', article_obj);
					
					var comments = [];
					for(var key in res.comments){
						res.comments[key].pub_date = new Date(res.comments[key].pub_date);
						if(!res.comments[key].reply_to){
							res.comments[key].replys = [];
							comments.push(res.comments[key]);
						}else{
							var reply_to = res.comments[key].reply_to;
							for(var loop in res.comments)
								if(res.comments[loop].id == reply_to){
									res.comments[key].reply_comment = res.comments[loop];
									break;
								}
							for(var cs in comments){
								if(comments[cs].id == reply_to){
									comments[cs].replys.push(res.comments[key]);
									break;
								}else{
									var replys = comments[cs].replys,
										is_find = false;
									for(var rs in replys){
										if(replys[rs].id == reply_to){
											replys.push(res.comments[key]);
											is_find = true;
											break;
										}
									}
									if(is_find)
										break;
								}
							}
						}
					}
					for(var key in comments){
						comments[key].n_replies = comments[key].replys.length;
					}
					comments.sort(function(a, b){
						return a.pub_date < b.pub_date;
					});
					for(var key in comments){
						comments[key].cindex = key;
					}
					self.$set('cAll', comments);
					window._all_comments = self.cAll;
					window._hot_comments = self.cHot;
				}
			});
		}
	}
</script>