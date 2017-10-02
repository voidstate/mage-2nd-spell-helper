<template>
	<div class="page">
		<div class="row sm-gutter">

			<div class="col-xs-12 col-sm-6 col-xl-4">

				<q-card>
					<q-card-title>
						What?
					</q-card-title>

					<q-card-separator/>

					<q-card-main>
						<p>Mage: the Awakening is a roleplaying game set in the Chronicles of Darkness by Onyx Path Publishing.</p>
						<p>You can read up on it <a href="http://whitewolf.wikia.com/wiki/Mage:_The_Awakening" target="_blank">here</a> or, more irreverently, <a href="https://1d4chan.org/wiki/Mage:_The_Awakening">here</a>.
						</p>
						<p>The game book is available in PDF or POD <a href="http://www.drivethrurpg.com/product/181754/Mage-the-Awakening-2nd-Edition">from Drive Thru RPG</a>.
						</p>
					</q-card-main>
				</q-card>
			</div>

			<div class="col-xs-12 col-sm-6 col-xl-4">

				<q-card>
					<q-card-title>
						Why?
					</q-card-title>

					<q-card-separator/>

					<q-card-main>
						<p>Laziness, mostly. </p>
						<p>Spellcasting in 2nd edition Mage is much more streamlined than before, but there are still quite a few elements to juggle - Arcana, Mana, dice pools, Paradox rolls, casting time, duration, potency, range, scale, withstand - plus various complications form individual spells, attainments, etc. It can be a lot for new players to absorb.</p>
						<p>Hopefully this app will help players not only work out the various pools but see how it all fits together.</p>
					</q-card-main>
				</q-card>
			</div>

			<div class="col-xs-12 col-sm-6 col-xl-4">
				<q-card>
					<q-card-title>
						Who?
					</q-card-title>

					<q-card-separator/>

					<q-card-main>
						<p>I'm a geek from Leeds, UK, who likes his coding job enough that he comes home and does it for fun, too.</p>
						<p>I've made some <a href="http://www.voidstate.com/rpg">other RPG stuff</a>, as well as <a href="http://xwing-builder.co.uk/build">a squadron builder for the X-Wing Miniatures Game</a> which is quite popular.
						</p>
					</q-card-main>
				</q-card>
			</div>

			<div class="col-xs-12 col-sm-6 col-xl-4">
				<q-card>
					<q-card-title>
						How?
					</q-card-title>

					<q-card-separator/>

					<q-card-main>
						<p>This app is pure JavaScript, using the <a href="https://vuejs.org/">Vue JS</a> and <a href="http://quasar-framework.org/">Quasar</a> frameworks, and <a href="http://ionicons.com/">Ionic icons</a>. You can view the source <a href="https://github.com/voidstate/mage-2nd-spell-helper">on Github</a>, if you want to contribute.
						</p>
					</q-card-main>
				</q-card>
			</div>
		</div>

		<div class="footer-row">
			<div class="col">
				<q-btn color="secondary" @click="$router.push('/')">
					Back
				</q-btn>
			</div>
		</div>
	</div>
</template>

<script>
	import {
		// Quasar components
		QCard, QCardTitle, QCardSeparator, QCardMain,
		QBtn,
		// Quasar utilities
		openURL,
	} from 'quasar'

	export default {
		name: 'spell-helper',
		components: {
			QCard, QCardTitle, QCardSeparator, QCardMain,
			QBtn
		},
		data()
		{
			return {}
		}
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

	.footer-row
		text-align right
		padding 16px
</style>
