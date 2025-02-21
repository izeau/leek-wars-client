<template lang="html">
	<div>
		<div class="page-header page-bar">
			<div>
				<h1>{{ $t('title') }}</h1>
				<v-menu offset-y lazy class="forum-language info">
					<div slot="activator">
						<img :src="forumLanguage.flag" class="flag">
						<img width="10" src="/image/selector.png">
					</div>
					<v-list :dense="true">
						<v-list-tile v-for="(language, i) in LeekWars.languages" :key="i" @click="setForumLanguage(language)">
							<v-list-tile-title class="language">
								<img :src="language.flag" class="flag">
								<span class="name">{{ language.name }}</span>
							</v-list-tile-title>
						</v-list-tile>
					</v-list>
				</v-menu>
			</div>
			<div class="tabs">
				<router-link to="/chat">
					<div class="tab action" icon="chat_bubble" link="/chat">
						<img src="image/icon/forum.png">
						<span>{{ $t('chat') }}</span>
					</div>
				</router-link>
				<div class="tab action disabled" icon="search" link="/search">
					<img class="search-icon" src="image/search.png" @click="search">
					<input v-model="searchQuery" type="text" @keyup.enter="search">
				</div>
			</div>
		</div>

		<panel class="first">
			<loader v-if="!categories" />
			<template v-else>
				<div v-if="!LeekWars.mobile" class="header category">
					<div class="seen"></div>
					<div class="text">{{ $t('category') }}</div>
					<div class="num-topics">{{ $t('topics') }}</div>
					<div class="num-messages">{{ $t('messages') }}</div>
				</div>
				<router-link v-ripple v-for="category in categories" :key="category.id" :to="'/forum/category-' + category.id" class="category">
					<div class="seen">
						<img v-if="category.seen" src="/image/forum_seen.png">
						<img v-else src="/image/forum_unseen.png">
					</div>
					<div class="text">
						<template v-if="category.type == 'normal'">
							<div class="title">{{ $t('category_' + category.name) }}</div>
							<div class="description">{{ $t('category_' + category.name + '_desc') }}</div>
						</template>
						<div v-else-if="category.type == 'team'">
							<div class="title">{{ category.name }}</div>
							<div class="description">{{ $t('team_forum_description') }}</div>
						</div>
						<div v-if="LeekWars.mobile" class="mobile-info">
							<span>{{ $t('n_topics', [LeekWars.formatNumber(category.topics)]) }} &nbsp;</span>
							<span>{{ $t('n_messages', [LeekWars.formatNumber(category.messages)]) }}</span>
						</div>
					</div>
					<div v-if="!LeekWars.mobile" class="num-topics">{{ category.topics | number }}</div>
					<div v-if="!LeekWars.mobile" class="num-messages">{{ category.messages | number }}</div>
				</router-link>
			</template>
		</panel>

		<panel>
			<h2 slot="title">
				<span v-if="connected_farmers.length">{{ $t('connected_farmers', [connected_farmers.length]) }}</span>
			</h2>
			<div slot="actions">
				<div class="button flat">
					<i v-if="expandFarmers" class="material-icons expand-connected-farmers" @click="expandFarmers = !expandFarmers">expand_more</i>
					<i v-else class="material-icons expand-connected-farmers" @click="expandFarmers = !expandFarmers">expand_less</i>
				</div>
			</div>
			<loader v-if="!connected_farmers.length" />
			<div v-else :class="{expanded: expandFarmers}" class="connected-farmers">
				<router-link v-for="(farmer, f) in connected_farmers" :key="farmer.id" :to="'/farmer/' + farmer.id">
					<span v-if="f > 0">, </span>
					<span :class="farmer.class">
						<rich-tooltip-farmer :id="farmer.id">{{ farmer.name }}</rich-tooltip-farmer>
					</span>
				</router-link>
			</div>
			<div class="grades-legend">
				{{ $t('legend') }} : <span class="admin">{{ $t('main.grade_admin') }}</span>,
				<span class="moderator">{{ $t('main.grade_moderator') }}</span>,
				<span class="contributor">{{ $t('main.grade_contributor') }}</span>,
				<span>{{ $t('main.grade_member') }}</span>
			</div>
		</panel>

		<panel class="last">
			<h2 slot="title">
				<router-link to="/chat">{{ $t('chat') }}</router-link>
				<v-menu offset-y lazy>
					<img slot="activator" :src="chatLanguage.flag" class="language-button">
					<v-list :dense="true">
						<v-list-tile v-for="(language, i) in LeekWars.languages" :key="i" @click="chatLanguage = language">
							<v-list-tile-title class="language">
								<img :src="language.flag" class="flag">
								<span class="name">{{ language.name }}</span>
							</v-list-tile-title>
						</v-list-tile>
					</v-list>
				</v-menu>
			</h2>
			<div slot="actions">
				<div v-if="!LeekWars.mobile" class="button flat" @click="LeekWars.addChat(chatLanguage.code, ChatType.GLOBAL, 'Chat ' + chatLanguage.code.toUpperCase())">
					<i class="material-icons">picture_in_picture_alt</i>
				</div>
			</div>
			<chat slot="content" :channel="chatLanguage.code" />
		</panel>
	</div>
</template>

<script lang="ts">
	import { ChatType } from '@/model/chat'
	import { Farmer } from '@/model/farmer'
	import { Language, LeekWars } from '@/model/leekwars'
	import { Component, Vue } from 'vue-property-decorator'
	
	@Component({ name: 'forum', i18n: {} })
	export default class Forum extends Vue {
		ChatType = ChatType
		categories: any = null
		connected_farmers: Farmer[] = []
		chatLanguage: Language | null = null
		forumLanguage: Language | null = null
		expandFarmers: boolean = false
		searchQuery: string = ''
		
		created() {
			const lang = localStorage.getItem('forum/language') as string || this.$i18n.locale
			this.forumLanguage = LeekWars.languages[lang]
			this.chatLanguage = LeekWars.languages[this.$i18n.locale]
			LeekWars.get('forum/get-categories/' + this.forumLanguage.code).then(data => {
				this.categories = data.categories
				this.$root.$emit('loaded')
			})
			LeekWars.get('farmer/get-connected').then(data => {
				this.connected_farmers = data.farmers
				LeekWars.setSubTitle(this.$t('forum.connected_farmers_subtitle', [data.count]))
			})
			LeekWars.setTitle(this.$t('forum.title'))
			LeekWars.setActions([
				{icon: 'chat_bubble', click: () => this.$router.push('/chat')},
				{icon: 'search', click: () => this.$router.push('/search')}
			])
		}
		setForumLanguage(language: Language) {
			this.forumLanguage = language
			this.categories = null
			localStorage.setItem('forum/language', language.code)
			LeekWars.get('forum/get-categories/' + language.code).then(data => {
				this.categories = data.categories
			})
		}
		search() {
			const query = this.searchQuery.replace(/ /g, '+')
			if (query) {
				this.$router.push('/search?query=' + query)
			} else {
				this.$router.push('/search')
			}
		}
	}
</script>

<style lang="scss" scoped>
	.forum-language {
		display: inline-block;
		padding: 0 4px;
		border-radius: 2px;
		cursor: pointer;
		vertical-align: bottom;
		img.flag {
			vertical-align: top;
			height: 32px;
		}
		img:not(.flag) {
			vertical-align: middle;
			margin-bottom: 3px;
			margin-left: 6px;
		}
	}
	.search-icon {
		cursor: pointer;
	}
	.panel.first /deep/ .content {
		padding: 5px;
	}
	.header.category {
		font-size: 17px;
		font-weight: 300;
		background-color: transparent;
		margin: 0;
	}
	.category {
		margin-bottom: 5px;
		display: flex;
		align-items: center;
	}
	.category:last-child {
		margin-bottom: 0;
	}
	.category > div {
		padding: 10px;
	}
	.category:not(.header) {
		border: 1px solid #ddd;
	}
	.category:not(.header):hover {
		background-color: white;
		box-shadow: 0px 2px 1px -1px rgba(0,0,0,0.2), 0px 1px 1px 0px rgba(0,0,0,0.14), 0px 1px 3px 0px rgba(0,0,0,0.12);
	}
	.category .seen {
		width: 40px;
		padding-top: 10px;
		padding-bottom: 8px;
		padding-right: 5px;
	}
	.category .seen img {
		height: 40px;
	}
	.category .text {
		flex: 1;
	}
	.category .title {
		font-weight: 300;
		font-size: 19px;
		margin-bottom: 4px;
		color: #333;
	}
	.category .description {
		color: #999;
		font-size: 14px;
	}
	.category .mobile-info {
		margin-top: 5px;
		color: #777;
		font-size: 14px;
	}
	.category .num-topics {
		flex: 0 0 80px;
		color: #777;
		text-align: center;
	}
	.category .num-messages {
		flex: 0 0 80px;
		color: #777;
		text-align: center;
	}
	.connected-farmers {
		height: 92px;
		overflow-y: hidden;
		text-align: justify;
	}
	.expand-connected-farmers {
		padding: 10px;
		cursor: pointer;
	}
	.connected-farmers.expanded {
		height: auto;
	}
	.grades-legend {
		font-size: 12px;
		text-align: right;
		padding-right: 10px;
		margin-top: 10px;
	}
	.chat {
		height: 400px;
	}
	.panel .language-button {
		height: 28px;
		max-height: 28px;
		max-width: none;
		padding: 5px;
		margin-left: 4px;
		margin-right: -10px;
		margin-top: -2px;
	}
	.flag {
		height: 28px;
	}
	.language {
		display: flex;
		align-items: center;
	}
	.language .name {
		padding-left: 8px;
	}
</style>

