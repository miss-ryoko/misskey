<template>
<div class="mk-user-preview">
	<template v-if="u != null">
		<div class="banner" :style="u.bannerUrl ? `background-image: url(${u.bannerUrl})` : ''"></div>
		<mk-avatar class="avatar" :user="u" :disable-preview="true"/>
		<div class="title">
			<router-link class="name" :to="u | userPage">{{ u | userName }}</router-link>
			<p class="username"><mk-acct :user="u"/></p>
		</div>
		<div class="description">{{ u.description }}</div>
		<div class="status">
			<div>
				<p>%i18n:@notes%</p><a>{{ u.notesCount }}</a>
			</div>
			<div>
				<p>%i18n:@following%</p><a>{{ u.followingCount }}</a>
			</div>
			<div>
				<p>%i18n:@followers%</p><a>{{ u.followersCount }}</a>
			</div>
		</div>
		<mk-follow-button v-if="$store.getters.isSignedIn && u.id != $store.state.i.id" :user="u"/>
	</template>
</div>
</template>

<script lang="ts">
import Vue from 'vue';
import * as anime from 'animejs';
import parseAcct from '../../../../../misc/acct/parse';

export default Vue.extend({
	props: {
		user: {
			type: [Object, String],
			required: true
		}
	},
	data() {
		return {
			u: null
		};
	},
	mounted() {
		if (typeof this.user == 'object') {
			this.u = this.user;
			this.$nextTick(() => {
				this.open();
			});
		} else {
			const query = this.user[0] == '@' ?
				parseAcct(this.user.substr(1)) :
				{ userId: this.user };

			(this as any).api('users/show', query).then(user => {
				this.u = user;
				this.open();
			});
		}
	},
	methods: {
		open() {
			anime({
				targets: this.$el,
				opacity: 1,
				'margin-top': 0,
				duration: 200,
				easing: 'easeOutQuad'
			});
		},
		close() {
			anime({
				targets: this.$el,
				opacity: 0,
				'margin-top': '-8px',
				duration: 200,
				easing: 'easeOutQuad',
				complete: () => this.$destroy()
			});
		}
	}
});
</script>

<style lang="stylus" scoped>
@import '~const.styl'

root(isDark)
	position absolute
	z-index 2048
	margin-top -8px
	width 250px
	background isDark ? #282c37 : #fff
	background-clip content-box
	border solid 1px rgba(#000, 0.1)
	border-radius 4px
	overflow hidden
	opacity 0

	> .banner
		height 84px
		background-color isDark ? #1c1e26 : #f5f5f5
		background-size cover
		background-position center

	> .avatar
		display block
		position absolute
		top 62px
		left 13px
		z-index 2
		width 58px
		height 58px
		border solid 3px isDark ? #282c37 : #fff
		border-radius 8px

	> .title
		display block
		padding 8px 0 8px 82px

		> .name
			display inline-block
			margin 0
			font-weight bold
			line-height 16px
			color isDark ? #fff : #656565

		> .username
			display block
			margin 0
			line-height 16px
			font-size 0.8em
			color isDark ? #606984 : #999

	> .description
		padding 0 16px
		font-size 0.7em
		color isDark ? #9ea4ad : #555

	> .status
		padding 8px 16px

		> div
			display inline-block
			width 33%

			> p
				margin 0
				font-size 0.7em
				color #aaa

			> a
				font-size 1em
				color $theme-color

	> .mk-follow-button
		position absolute
		top 92px
		right 8px

.mk-user-preview[data-darkmode]
	root(true)

.mk-user-preview:not([data-darkmode])
	root(false)

</style>
