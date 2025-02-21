<template>
	<v-menu :close-on-content-click="false" :disabled="disabled" :max-width="expand_items ? 600 : 360" :nudge-top="bottom ? 0 : 6" :open-delay="_open_delay" :close-delay="_close_delay" :top="!bottom" :bottom="bottom" open-on-hover offset-y lazy @input="open($event)">
		<slot slot="activator"></slot>
		<div v-if="content_created" :class="{expanded: expand_items}" class="card">
			<loader v-if="!leek" :size="30" />
			<template v-else>
				<div class="flex">
					<router-link :to="'/leek/' + leek.id">
						<div class="leek-image">
							<leek-image :leek="leek" :scale="0.3" />
						</div>
					</router-link>
					<div class="info">
						<span class="name">
							<router-link :to="'/leek/' + leek.id" class="text">{{ leek.name }}</router-link>
							<router-link :to="'/farmer/' + leek.farmer.id">
								<avatar :farmer="leek.farmer" :title="leek.farmer.name" />
							</router-link>
							<router-link v-if="leek.team" :to="'/team/' + leek.team.id">
								<emblem :team="leek.team" :title="leek.team.name" />
							</router-link>
						</span>
						<talent :talent="leek.talent" />
						<span class="talent-more">({{ leek.talent_more >= 0 ? '+' + leek.talent_more : leek.talent_more }})</span>
						<span class="level">• {{ $t('main.level_n', [leek.level]) }}</span>
						<v-btn class="expand" icon small @click="expand_items = !expand_items">
							<v-icon v-if="expand_items">expand_less</v-icon>
							<v-icon v-else>expand_more</v-icon>
						</v-btn>
					</div>
				</div>
				<div v-if="expand_items">
					<table class="leeks">
						<tr>
							<th>{{ $t('main.name') }}</th>
							<th>{{ $t('main.level') }}</th>
							<th><img src="/image/talent.png"></th>
							<th v-for="c in LeekWars.characteristics" :key="c" class="c"><img :src="'/image/charac/small/' + c + '.png'" :class="{zero: leek[c] === 0}"></th>
						</tr>
						<tr>
							<td class="leek-name"><router-link :to="'/leek/' + leek.id">{{ leek.name }}</router-link></td>
							<td>{{ leek.level }}</td>
							<td><b>{{ leek.talent }}</b></td>
							<td v-for="c in LeekWars.characteristics" :key="c" :class="['color-' + c, leek[c] === 0 ? 'zero' : '']" class="c">{{ leek[c] }}</td>
						</tr>
					</table>
					<div class="items">
						<div class="weapons">
							<tooltip v-for="weapon in leek.orderedWeapons" :key="weapon.id">
								<img slot="activator" :src="'/image/weapon/' + LeekWars.weapons[weapon.template].name + '.png'" class="weapon">
								<b>{{ $t('weapon.' + LeekWars.weapons[weapon.template].name) }}</b>
								<br>
								{{ $t('main.level_n', [LeekWars.weapons[weapon.template].level]) }}
								<br>
								<small>{{ 'WEAPON_' + LeekWars.weapons[weapon.template].name.toUpperCase() }}</small>
							</tooltip>
						</div>
						<div class="chips">
							<tooltip v-for="chip in leek.orderedChips" :key="chip.id">
								<img slot="activator" :src="'/image/chip/small/' + LeekWars.chips[chip.template].name + '.png'" class="chip">
								<b>{{ $t('chip.' + LeekWars.chips[chip.template].name) }}</b>
								<br>
								{{ $t('main.level_n', [LeekWars.chips[chip.template].level]) }}
								<br>
								<small>{{ 'CHIP_' + LeekWars.chips[chip.template].name.toUpperCase() }}</small>
							</tooltip>
						</div>
					</div>
				</div>
			</template>
		</div>
	</v-menu>
</template>

<script lang="ts">
	import { Leek } from '@/model/leek'
	import { LeekWars } from '@/model/leekwars'
	import { store } from '@/model/store'
	import { Component, Prop, Vue } from 'vue-property-decorator'
	@Component({})
	export default class RichTooltipLeek extends Vue {
		@Prop({required: true}) id!: number
		@Prop() disabled!: boolean
		@Prop() bottom!: boolean
		@Prop() instant!: boolean
		content_created: boolean = false
		leek: Leek | null = null
		expand_items: boolean = false

		get _open_delay() {
			return this.instant ? 0 : 200
		}
		get _close_delay() {
			return this.instant ? 0 : 200
		}
		open(v: boolean) {
			if (!v) {
				this.expand_items = false
			}
			this.content_created = true
			if (this.id > 0 && !this.leek) {
				LeekWars.get<Leek>('leek/rich-tooltip/' + this.id).then(leek => {
					this.leek = new Leek(leek)
				})
			}
		}
	}
</script>

<style lang="scss" scoped>
	.card {
		padding: 8px;
		height: 65px;
	}
	.card.expanded {
		height: auto;
	}
	.leek-image {
		display: flex;
		align-items: flex-end;
		max-height: 65px;
		svg {
			min-width: 45px;
			min-height: 65px;
		}
	}
	.spacer {
		flex: 1;
	}
	.info {
		flex: 1;
		padding-left: 10px;
		.icon {
			width: 15px;
			margin-right: 4px;
			vertical-align: middle;
			padding-bottom: 2px;
		}
		.stat {
			padding-right: 4px;
			font-size: 13px;
			img {
				opacity: 0.5;
			}
		}
	}
	.name {
		display: flex;
		align-items: center;
		font-size: 16px;
		height: 25px;
		margin-right: -4px;
		margin-bottom: 6px;
		img {
			width: 17px;
			height: 17px;
			margin-right: 3px;
		}
		i {
			font-size: 18px;
		}
		.avatar {
			margin-left: 5px;
			margin-top: 3px;
		}
		.emblem {
			margin-left: 5px;
			margin-top: 3px;
		}
		.country {
			margin-left: 5px;
			margin-top: 1px;
		}
	}
	.talent-more {
		font-size: 15px;
		margin-left: 5px;
		color: #888;
		display: inline-block;
		vertical-align: top;
		margin-top: 10px;
	}
	.level {
		display: inline-block;
		font-size: 15px;
		font-weight: 500;
		margin-left: 5px;
		vertical-align: top;
		margin-top: 10px;
		color: #555;
	}
	.expand {
		vertical-align: top;
		margin-top: 5px;
		margin-left: 10px;
	}
	.leeks {
		text-align: left;
		width: 600px;
		margin: 0 -8px;
		tr {
			border-bottom: 1px solid #ddd;
		}
		tr:nth-child(2n) {
			background: #f7f7f7;
		}
		td, th {
			padding: 3px 4px;
		}
		td:first-child, th:first-child {
			padding-left: 10px;
			padding-right: 10px;
		}
		img {
			width: 18px;
		}
		.c {
			width: 30px;
			font-weight: 500;
		}
		.zero {
			filter: saturate(0);
			opacity: 0.3;
		}
		.leek-name {
			max-width: 120px;
			text-overflow: ellipsis;
			overflow: hidden;
		}
	}
	.items {
		display: flex;
		margin: 0 -8px;
		margin-bottom: -8px;
		align-items: center;
		width: 600px;
	}
	.weapons {
		flex: 1.1;
		padding: 10px;
		.weapon {
			width: 120px;
			max-height: 46px;
			margin: 8px;
			vertical-align: middle;
			object-fit: contain;
		}
	}
	.chips {
		flex: 1;
		padding: 4px;
		.chip {
			width: 40px;
			margin: 2px;
			vertical-align: top;
		}
	}
</style>