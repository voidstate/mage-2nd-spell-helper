<template>
		<div class="fieldset">
			<h6><b>{{ name }}</b> (+{{ bonus }})</h6>
			<p class="text-muted">
				<small>{{ desc }}</small>
			</p>
			<q-btn small flat @click="deleteYantra()" class="closeBtn">
				<q-icon name="ion-ios-close"/>
			</q-btn>


			<div v-if="canBeDedicatedTool">
				<q-field helper="A dedicated tool may also be a Path or Order tool. Dedicated Tools provide -2 Paradox dice." class="dedicated-tool-field">
					<p>
						<small>Is this {{ name }} also a Dedicated Tool?</small>
						<br>
						<q-toggle checked-icon="ion-ios-checkmark-outline" unchecked-icon="ion-ios-close-outline" v-model="isDedicatedTool" :val="true"/>
					</p>

				</q-field>
			</div>
		</div>
</template>

<script>
	import {
		// Quasar components
		QBtn,
		QIcon,
		QField,
		QToggle

	} from 'quasar'

	export default {
		name: 'my-yantra',
		components: {
			QBtn,
			QIcon,
			QField,
			QToggle
		},
		props: [ 'yantraKey', 'name', 'desc', 'bonus', 'isDedicatedTool' ],
		data()
		{
			return {}
		},
		computed: {
			canBeDedicatedTool()
			{
				return [ 't2', 't3' ].includes( this.yantraKey )
			}
		},
		watch: {
			isDedicatedTool()
			{
				this.$emit( 'UpdateIsDedicatedTool', this.yantraKey, this.isDedicatedTool )
			}
		},
		methods: {
			deleteYantra()
			{
				this.$emit( 'delete', this.yantraKey )
			}
		}
	}
</script>

<style lang="stylus" scoped>
	.fieldset
		position relative

	.dedicated-tool-field
		margin-top 0

	.closeBtn
		position absolute
		top -5px
		right 0

</style>
