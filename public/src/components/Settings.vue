<template>
	<div class="page">
		<div class="row sm-gutter">

			<div class="col-xs-12 col-sm-6 col-xl-4">

				<q-card>
					<q-card-title>
						Spell Helper Settings
					</q-card-title>

					<q-card-separator/>

					<q-card-main>

						<q-list link>
							<q-item tag="label" multiline>
								<q-item-main>
									<q-item-tile label>Show Attainments</q-item-tile>
									<q-item-tile sublabel>Attainments allow the caster to improve spell factors</q-item-tile>
								</q-item-main>
								<q-item-side right>
									<q-toggle v-model="settings.showAttainments"></q-toggle>
								</q-item-side>
							</q-item>

						</q-list>

					</q-card-main>
				</q-card>
			</div>

		</div>
	</div>
</template>

<script>
	import {
		// Quasar components
		QCard, QCardTitle, QCardSeparator, QCardMain,
		QList, QItem, QItemMain, QItemTile, QItemSide,
		QToggle,
		// Quasar utilities
		openURL,
	} from 'quasar'

	export default {
		name: 'spell-helper',
		components: {
			QCard, QCardTitle, QCardSeparator, QCardMain,
			QList, QItem, QItemMain, QItemTile, QItemSide,
			QToggle
		},
		props: [
			'settings'
		]
	}
</script>

<style lang="stylus">
	@import '~variables'

	.page
		padding 24px

	.q-card-primary
		background-color $primary
		color $white

	.q-card-title
		font-size 18px
		text-transform uppercase
		> .q-icon
			font-size 36px
			margin-top -5px
			margin-right 10px
			&.smaller
				font-size 32px
</style>
