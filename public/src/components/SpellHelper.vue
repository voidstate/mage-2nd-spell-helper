<template>
	<q-layout>

		<!-- Header -->
		<q-toolbar slot="header" color="primary">

			<q-toolbar-title @click="$router.push('/')">
				<a @click="$router.push('/about')"> Improvised Spell Helper </a>
				<span slot="subtitle">Mage: The Awakening 2nd Edition</span>

			</q-toolbar-title>

			<q-btn flat @click="$router.push('/about')">
				<q-icon name="ion-ios-information"/>
			</q-btn>
			<q-btn flat @click="$refs.settingsModal.open()">
				<q-icon name="ion-ios-gear"/>
			</q-btn>
			<div class="toolbar-spacer gt-sm">|</div>
			<q-btn flat @click="open('mailto:rpg@voidstate.com')">
				<q-icon name="ion-ios-email"/>
				<q-tooltip>
					<small>Bugs, typos, suggestions, buy me a beer? Drop me a line.</small>
				</q-tooltip>
			</q-btn>
			<q-btn flat @click="open('https://github.com/voidstate/mage-2nd-spell-helper')">
				<q-icon name="ion-social-github"/>
				<q-tooltip>
					<small>Visit Github repo</small>
				</q-tooltip>
			</q-btn>
		</q-toolbar>

		<!-- Stepper -->
		<q-toolbar slot="footer" class="fixed-bottom footer">
			<q-toolbar-title>
				<q-chip icon="ion-ios-bolt-outline" id="footer-reach-chip" class="info" small>
					<span class="footer-chip-inner">
						<span class="footer-chip-label gt-xs">Reach</span> {{ usedReach }}/{{ freeReach }}
					</span>
				</q-chip>
				<q-chip icon="ion-ios-analytics-outline" id="footer-dice-pool-chip" class="info" :class="{ warning: isDicePoolTooLow }" small>
					<span class="footer-chip-inner">
						<span class="footer-chip-label gt-xs">Dice pool</span> {{ dicePool }}
					</span>
					<q-tooltip v-if="isDicePoolTooLow" class="warning">
						If the dice pool, after Yantras, is -6 or less, the spell is impossible
					</q-tooltip>
				</q-chip>
				<q-chip icon="ion-ios-pulse" id="footer-mana-chip" class="info" small>
					<span class="footer-chip-inner">
						<span class="footer-chip-label gt-xs">Mana</span> {{ totalMana }}
					</span>
				</q-chip>
				<q-chip icon="ion-ios-flower" id="footer-paradox-chip" class="warning" small :class="{ hidden: !hasParadox }">
					<span class="footer-chip-inner">
						<span class="footer-chip-label gt-xs">Paradox</span> {{ paradoxDice }}
					</span>
					<q-tooltip class="warning">
						You have exceeded the free reach from your Arcanum and roll for Paradox
					</q-tooltip>
				</q-chip>
			</q-toolbar-title>
		</q-toolbar>

		<!-- Footer -->
		<q-stepper color="secondary" ref="stepper" alternative-labels contractable v-model="currentStep">

			<q-step default name="first" title="Spell Overview" icon="ion-ios-star-outline" active-icon="ion-ios-star-outline" done-icon="ion-ios-star-outline">

				<h5>
					Spell Overview <br class="lt-md">
					<small>Details on the caster, spell and subject</small>
				</h5>

				<div class="row sm-gutter">

					<!-- Caster -->
					<div class="col-xs-12 col-sm-6 col-xl-4">

						<q-card>
							<q-card-title>
								Caster
							</q-card-title>

							<q-card-separator/>

							<q-card-main>

								<div class="fieldset">
									<h6>Gnosis</h6>
									<q-slider v-model="caster.gnosis" :min="1" :max="10" :step="1" label-always snap/>
								</div>

								<div class="fieldset"><h6>Highest Arcanum Used in the Spell</h6>
									<q-select v-model="caster.arcanaName" :options="arcanaNameOptions"/>
								</div>

								<div class="fieldset">
									<h6>Mage's {{ caster.arcanaName }} Arcanum</h6>
									<q-slider v-model="caster.arcana" :min="1" :max="5" :step="1" label-always snap/>
								</div>

								<div class="fieldset">
									<q-field helper="Gnosis provides different limits for the highest and non-highest Arcana.">
										<h6>Is {{ caster.arcanaName }} the Mage's Highest Arcanum?</h6>
										<q-toggle checked-icon="ion-ios-checkmark-outline" unchecked-icon="ion-ios-close-outline" v-model="caster.isHighestArcana"/>
									</q-field>
								</div>

								<div class="fieldset">
									<q-field helper="Spell from a Common or Inferior arcanum cost one Mana to cast (unless the spell is a Praxis).">
										<h6>Is {{ caster.arcanaName }} One of the Mage's Ruling Arcana?</h6>
										<q-toggle checked-icon="ion-ios-checkmark-outline" unchecked-icon="ion-ios-close-outline" v-model="caster.isRulingArcana"/>
									</q-field>
								</div>

								<div class="fieldset">
									<q-field helper="Maintaining more spells than the caster's Gnosis adds Reach">
										<h6>Active Spells</h6>
										<q-input v-model="caster.activeSpells" type="number" :min="0"/>
									</q-field>
								</div>

							</q-card-main>
						</q-card>

					</div>

					<!-- Spell details -->
					<div class="col-xs-12 col-sm-6 col-xl-4">

						<q-card>
							<q-card-title>
								Spell
							</q-card-title>

							<q-card-separator/>

							<q-card-main>

								<div class="fieldset">
									<h6>Spell's Required {{ caster.arcanaName }} Arcanum</h6>
									<q-slider v-model="spell.arcana" :min="1" :max="5" :step="1" label-always snap stack-label="test"/>
								</div>

								<div class="fieldset">
									<q-field helper="The Primary Factor is boosted up to the caster's Arcanum rating without incurring a penalty.">
										<h6>Primary Factor</h6>
										<q-select v-model="spell.primaryFactor" :options="spellFactorOptions"/>
									</q-field>
								</div>

								<div class="fieldset">
									<h6>Is the Spell a Rote?</h6>
									<q-toggle checked-icon="ion-ios-checkmark-outline" unchecked-icon="ion-ios-close-outline" v-model="spell.isRote" :disable="roteOrPraxis === 'praxis'"/>
									<q-tooltip class="warning" v-show="roteOrPraxis === 'praxis'">
										You may only choose either Rote or Praxis.
									</q-tooltip>
								</div>

								<div :class="{ hidden: !spell.isRote }" class="fieldset">
									<q-field helper="Skill dots are  added to the dice pool as a Yantra bonus.">
										<h6>Rote Skill</h6>
										<q-slider v-model="spell.roteSkill" :min="1" :max="10" :step="1" label-always snap/>
									</q-field>
								</div>

								<div class="fieldset">
									<q-field helper="Praxes only require three successes for an exceptional success, and do not require a Mana if the spell is from a Common or Inferior Arcanum.">
										<h6>Is the Spell a Praxis?</h6>
										<q-toggle checked-icon="ion-ios-checkmark-outline" unchecked-icon="ion-ios-close-outline" v-model="spell.isPraxis" :disable="roteOrPraxis==='rote'"/>
										<q-tooltip class="warning" v-show="roteOrPraxis === 'rote'">
											You may only choose either Rote or Praxis.
										</q-tooltip>
									</q-field>
								</div>

								<div class="fieldset">
									<q-field helper="From Fate magic, Merits, Artifacts, etc.">
										<h6>Additional Spellcasting Dice</h6>
										<q-input v-model="spell.bonusDice" type="number" :min="0"/>
									</q-field>
								</div>

								<div class="fieldset">
									<q-field helper="Adds +3 dice">
										<h6>Spend Willpower?</h6>
										<q-toggle checked-icon="ion-ios-checkmark-outline" unchecked-icon="ion-ios-close-outline" v-model="spell.spendWillpower"/>
									</q-field>
								</div>
							</q-card-main>
						</q-card>

					</div>

					<!-- Subject -->
					<div class="col-xs-12 col-sm-6 col-xl-4">

						<q-card>
							<q-card-title>
								Subject
							</q-card-title>

							<q-card-separator/>

							<q-card-main>

								<div class="fieldset">
									<h6>Is the Spell Resisted?</h6>
									<q-toggle checked-icon="ion-ios-checkmark-outline" unchecked-icon="ion-ios-close-outline" v-model="subject.isResisted"/>
								</div>

								<div :class="{ hidden: !subject.isResisted }" class="fieldset">
									<q-field helper="Potency must exceed this level for the spell to take effect.">
										<h6>Withstand Rating</h6>
										<q-slider v-model="subject.withstand" :min="1" :max="10" :step="1" label-always snap/>
									</q-field>
								</div>

								<div :class="{ hidden: !subject.isResisted }" class="fieldset">
									<q-field helper="If a spell has multiple Withstand ratings (eg. a Withstood spell cast with a Sympathetic Range Attainment or through the Gauntlet), it uses the highest rating, +1 for every additional rating.">
										<h6>Number of Withstand Ratings</h6>
										<q-input v-model="subject.numWithstands" type="number" :min="1"/>
									</q-field>
								</div>

							</q-card-main>
						</q-card>

					</div>

				</div>

				<q-stepper-navigation>
					<q-btn color="secondary" flat @click="resetData()">Reset</q-btn>
					<q-btn color="secondary" @click="stepNext()">Next</q-btn>
				</q-stepper-navigation>
			</q-step>

			<!-- 2: Factors -->
			<q-step name="second" title="Factors" icon="ion-ios-toggle" active-icon="ion-ios-toggle" done-icon="ion-ios-toggle">
				<h5>
					Spell Factors <br class="lt-md">
					<small>All spell factors start at Standard. Spend Reach to increase them to Advanced. Take penalties to increase spell factors that allow it.</small>
				</h5>

				<div class="row sm-gutter">

					<!-- Casting time -->
					<div class="col-xs-12 col-sm-6 col-xl-4">

						<q-card>
							<q-card-title>
								<q-icon name="ion-ios-stopwatch" class="smaller"/>
								Casting Time
							</q-card-title>

							<q-card-separator/>

							<q-card-main>
								<q-list>
									<q-collapsible label="Standard" group="casting-time" :opened="!isAdvanced('castingTime')">
										<p>
											<b>Ritual Casting Time</b>
										</p>
										<div v-for="castingTime in standardCastingTimeOptions">
											<q-radio v-model="spell.factors.castingTime" :val="castingTime.key" :label="castingTime.label"/>
										</div>
									</q-collapsible>

									<q-collapsible label="Advanced" group="casting-time" :opened="isAdvanced('castingTime')">

										<q-field helper="Using more than one Yantra (or High Speech) will increase this time.">
											<label>
												<q-radio v-model="spell.factors.castingTime" val="a1"/>
												<b>Quick casting time:</b>1 Turn </label>
										</q-field>

										<q-list link class="attainments" v-show="settings.showAttainments">
											<q-list-header>Attainments</q-list-header>

											<q-item multiline tag="label">
												<q-item-main>
													<q-item-tile label>
														Time in a Bottle (Time <span class="arcana-dot">&#9679;&#9679;&#9679;&#9679;</span>)
													</q-item-tile>
													<q-item-tile sublabel>
														Advanced Casting Time costs 1 Mana instead of 1 Reach.
													</q-item-tile>
												</q-item-main>
												<q-item-side right>
													<q-toggle checked-icon="ion-ios-checkmark-outline" unchecked-icon="ion-ios-close-outline" v-model="spell.attainments.timeInABottle"/>
												</q-item-side>
											</q-item>
										</q-list>

									</q-collapsible>

								</q-list>
							</q-card-main>
						</q-card>

					</div>

					<!-- Duration -->
					<div class="col-xs-12 col-sm-6 col-xl-4" :class="{'order-first': spell.primaryFactor === 'duration'}">

						<q-card :class="{'main-card': spell.primaryFactor === 'duration'}">

							<q-card-title>
								<q-icon name="ion-ios-clock" class="smaller"/>
								Duration
								<q-chip class="main-card-chip pull-right" :class="{ hidden: spell.primaryFactor !== 'duration' }">
									<q-icon name="ion-ios-medical"/>
									<span class="gt-xs">Primary Factor</span>
									<q-tooltip class="lt-sm">
										Primary Factor
									</q-tooltip>
								</q-chip>
							</q-card-title>

							<q-card-separator/>

							<q-card-main>
								<q-list>
									<q-collapsible label="Standard" group="duration" :opened="!isAdvanced('duration')">

										<div v-for="duration in standardDurationOptions" :key="duration.key">
											<q-radio v-model="spell.factors.duration" :val="duration.key" :label="duration.label"/>
										</div>

									</q-collapsible>

									<q-collapsible label="Advanced" group="duration" :opened="isAdvanced('duration')">

										<div v-for="duration in advancedDurationOptions" :key="duration.key">
											<q-radio v-model="spell.factors.duration" :val="duration.key" :label="duration.label"/>
										</div>

										<q-list link class="attainments" v-show="settings.showAttainments && caster.arcanaName === 'Matter'">
											<q-list-header>Attainments</q-list-header>
											<q-item multiline tag="label">
												<q-item-main>
													<q-item-tile label>
														Permanence (Matter <span class="arcana-dot">&#9679;&#9679;</span>)
													</q-item-tile>
													<q-item-tile sublabel>
														Advanced Scale costs 1 Mana instead of 1 Reach.
													</q-item-tile>
												</q-item-main>
												<q-item-side right>
													<q-toggle checked-icon="ion-ios-checkmark-outline" unchecked-icon="ion-ios-close-outline" v-model="spell.attainments.permanence"/>
												</q-item-side>
											</q-item>
										</q-list>

									</q-collapsible>

								</q-list>
							</q-card-main>
						</q-card>

					</div>

					<!-- Potency -->
					<div class="col-xs-12 col-sm-6 col-xl-4" :class="{'order-first': spell.primaryFactor === 'potency'}">

						<q-card :class="{'main-card': spell.primaryFactor === 'potency'}">
							<q-card-title>
								<q-icon name="ion-ios-flame"/>
								Potency
								<q-chip class="main-card-chip pull-right" :class="{ hidden: spell.primaryFactor !== 'potency' }">
									<q-icon name="ion-ios-medical"/>
									<span class="gt-xs">Primary Factor</span>
									<q-tooltip class="lt-sm">
										Primary Factor
									</q-tooltip>
								</q-chip>
							</q-card-title>

							<q-card-separator/>

							<q-card-main>

								<div class="alert-bar alert-bar-warning" :class="{ hidden: totalWithstand === 0 }">
									<q-icon name="ion-android-warning"/>
									Potency must exceed the subject's Withstand of {{ totalWithstand }}
								</div>

								<q-list>

									<q-collapsible label="Standard" group="potency" :opened="!isAdvanced('potency')">
										<div>
											<div v-for="potency in standardPotencyOptions" :key="potency.key">
												<q-radio v-model="spell.factors.potency" :val="potency.key" :label="potency.label" :disable="potency.value <= totalWithstand"/>
												<q-tooltip class="warning" v-show="potency.value <= totalWithstand">
													Potency must exceed the subject's Withstand of {{ totalWithstand }}
												</q-tooltip>
											</div>
										</div>
									</q-collapsible>

									<q-collapsible label="Advanced" group="potency" :opened="isAdvanced('potency')">
										<div>
											<q-field helper="Advanced Potency grants an additional -2 to Withstand.">
												<div v-for="potency in advancedPotencyOptions" :key="potency.key">
													<q-radio v-model="spell.factors.potency" :val="potency.key" :label="potency.label" :disable="potency.value <= ( totalWithstand - 2 )"/>
													<q-tooltip class="warning" v-show="potency.value <= ( totalWithstand - 2 )">
														Potency must exceed the subject's Withstand of {{ totalWithstand - 2 }}
													</q-tooltip>
												</div>
											</q-field>
										</div>
									</q-collapsible>

								</q-list>
							</q-card-main>
						</q-card>

					</div>

					<!-- Range -->
					<div class="col-xs-12 col-sm-6 col-xl-4">

						<q-card>
							<q-card-title>
								<q-icon name="ion-ios-eye"/>
								Range
							</q-card-title>

							<q-card-separator/>

							<q-card-main>
								<q-list>

									<q-collapsible label="Standard" group="range" :opened="!isAdvanced('range')">
										<label>
											<q-radio v-model="spell.factors.range" val="s1"/>
											<b>Standard:</b> Self/touch or Aimed </label>
									</q-collapsible>

									<q-collapsible label="Advanced" group="range" :opened="isAdvanced('range')">
										<p>
											<label>
												<q-radio v-model="spell.factors.range" val="a1"/>
												<b>Advanced:</b> Sensory </label>
										</p>

										<q-list link class="attainments" v-show="settings.showAttainments">
											<q-list-header>Attainments</q-list-header>

											<q-item multiline tag="label">
												<q-item-main>
													<q-item-tile label>
														Sympathetic Range (Space <span class="arcana-dot">&#9679;&#9679;</span>)
													</q-item-tile>
													<q-item-tile sublabel>
														Subject can be beyond sensory range. Requires Advanced Range, a sympathy Yantra and costs +1 Mana.
													</q-item-tile>
												</q-item-main>
												<q-item-side right>
													<q-toggle checked-icon="ion-ios-checkmark-outline" unchecked-icon="ion-ios-close-outline" v-model="spell.attainments.sympatheticRange"/>
												</q-item-side>
											</q-item>

											<q-item multiline tag="label">
												<q-item-main>
													<q-item-tile label>
														Temporal Sympathy (Time <span class="arcana-dot">&#9679;&#9679;</span>)
													</q-item-tile>
													<q-item-tile sublabel>
														Cast a spell at subject's past self. Requires Advanced Range, a sympathy Yantra and costs +1 Mana. Can only be used with Time spells that allow it or spells combined with them.
													</q-item-tile>
												</q-item-main>
												<q-item-side right>
													<q-toggle checked-icon="ion-ios-checkmark-outline" unchecked-icon="ion-ios-close-outline" v-model="spell.attainments.temporalSympathy"/>
												</q-item-side>
											</q-item>

										</q-list>

									</q-collapsible>

								</q-list>
							</q-card-main>
						</q-card>

					</div>

					<!-- Scale -->
					<div class="col-xs-12 col-sm-6 col-xl-4">

						<q-card>

							<q-card-title icon="ion-ios-people-outline">
								<q-icon name="ion-ios-people"/>
								Scale
							</q-card-title>

							<q-card-separator/>

							<q-card-main>
								<q-list>
									<q-collapsible label="Standard" group="scale" :opened="!isAdvanced('scale')">

										<table class="q-table highlight responsive compact">
											<thead>
											<tr>
												<th></th>
												<th class="text-left">Number of Subjects</th>
												<th class="text-left">Size of Largest Subject</th>
												<th class="text-left">Area of Effect</th>
												<th class="text-left">Dice Penalty</th>
											</tr>
											</thead>
											<tbody>
											<tr v-for="scale in standardScaleOptions" :key="scale.key">
												<td>
													<q-radio v-model="spell.factors.scale" :val="scale.key"/>
												</td>
												<td data-th="Number of Subjects">{{ scale.number }}</td>
												<td data-th="Size of Largest Subject">{{ scale.size }}</td>
												<td data-th="Area of Effect">
													<small>{{ scale.area }}</small>
												</td>
												<td data-th="Dice Penalty">{{ scale.penalty }} dice</td>
											</tr>
											</tbody>
										</table>

									</q-collapsible>

									<q-collapsible label="Advanced" group="scale" :opened="isAdvanced('scale')">
										<q-field helper="Additional scale doubles the number of subjects and adds 5 size per additional -2 dice penalty.">
											<table class="q-table highlight responsive compact">
												<thead>
												<tr>
													<th></th>
													<th class="text-left">Number of Subjects</th>
													<th class="text-left">Size of Largest Subject</th>
													<th class="text-left">Area of Effect</th>
													<th class="text-left">Dice Penalty</th>
												</tr>
												</thead>
												<tbody>
												<tr v-for="scale in advancedScaleOptions" :key="scale.key">
													<td>
														<q-radio v-model="spell.factors.scale" :val="scale.key"/>
													</td>
													<td data-th="Number of Subjects">{{ scale.number }}</td>
													<td data-th="Size of Largest Subject">{{ scale.size }}</td>
													<td data-th="Area of Effect" class="small">{{ scale.area }}</td>
													<td data-th="Dice Penalty">{{ scale.penalty }} dice</td>
												</tr>
												</tbody>
											</table>
										</q-field>

										<q-list link class="attainments" v-show="settings.showAttainments">
											<q-list-header>Attainments</q-list-header>

											<q-item multiline tag="label">
												<q-item-main>
													<q-item-tile label>
														Everywhere (Space <span class="arcana-dot">&#9679;&#9679;&#9679;&#9679;</span>)
													</q-item-tile>
													<q-item-tile sublabel>
														Advanced Scale costs 1 Mana instead of 1 Reach
													</q-item-tile>
												</q-item-main>
												<q-item-side right>
													<q-toggle checked-icon="ion-ios-checkmark-outline" unchecked-icon="ion-ios-close-outline" v-model="spell.attainments.everywhere"/>
												</q-item-side>
											</q-item>
										</q-list>

									</q-collapsible>

								</q-list>
							</q-card-main>
						</q-card>

					</div>

					<!-- Extra reach -->
					<div class="col-xs-12 col-sm-6 col-xl-4">

						<q-card>
							<q-card-title>
								<q-icon name="ion-ios-plus" class="smaller"/>
								Extra Reach
							</q-card-title>
							<q-card-separator/>
							<q-card-main>
								<q-input v-model="spell.extraReach" type="number" float-label="Reach from spell effects" :min="0"/>
							</q-card-main>
						</q-card>

					</div>
				</div>

				<q-stepper-navigation>
					<q-btn color="secondary" flat @click="stepPrevious()">Back</q-btn>
					<q-btn color="secondary" @click="stepNext()">Next</q-btn>
				</q-stepper-navigation>
			</q-step>


			<!-- 3: Yantras -->
			<q-step name="third" title="Yantras" icon="ion-ios-color-wand" active-icon="ion-ios-color-wand" done-icon="ion-ios-color-wand">

				<h5>
					Yantras <br class="lt-md">
					<small>Use locations, actions, and tools to help form the spell's Imago.</small>
				</h5>

				<div class="alert-bar alert-bar-info">
					<q-icon name="ion-ios-information"/>
					Gnosis {{ caster.gnosis }} allows the use of {{ maxYantras }} yantras
				</div>

				<div class="row sm-gutter">

					<div class="col-sm-12 col-md-9 col-lg-6">

						<q-card>
							<q-card-title>
								Yantras
							</q-card-title>

							<q-card-separator/>

							<q-card-main>

								<div>
									<my-yantra v-for="yantra in spell.yantras" key="yantra.yantraKey" v-bind="yantra" @delete="deleteYantra" @UpdateIsDedicatedTool="updateYantraIsDedicatedTool"></my-yantra>
								</div>

								<div class="text-right">
									<q-btn color="secondary" icon="ion-ios-color-wand" @click="$refs.yantrasModal.open()">
										Add a Yantra
									</q-btn>
								</div>
							</q-card-main>
						</q-card>

					</div>

				</div>

				<q-stepper-navigation>
					<q-btn color="secondary" flat @click="stepPrevious()">Back</q-btn>
					<q-btn color="secondary" @click="stepNext()">Next</q-btn>
				</q-stepper-navigation>
			</q-step>

			<q-step name="fourth" title="Paradox" icon="ion-ios-flower" active-icon="ion-ios-flower" done-icon="ion-ios-flower">
				<h5>
					Paradox <br class="lt-md">
					<small>Using too much Reach runs the risk of complicating a spell to the point of warping it into a Paradox.</small>
				</h5>

				<div v-show="hasParadox" class="alert-bar alert-bar-warning">
					<q-icon name="ion-alert-circled"/>
					This spell will require a Paradox roll.
				</div>

				<div class="row sm-gutter">

					<div class="col-xs-12 col-sm-6">

						<q-card>
							<q-card-title>
								Situational Modifiers
							</q-card-title>

							<q-card-separator/>

							<q-card-main>
								<div class="fieldset">
									<q-field helper="A mage is Inured once they no longer risk losing Wisdom from a spell.">
										<h6>Is the Mage's Inured to this Spell? (+2 dice)</h6>
										<q-toggle checked-icon="ion-ios-checkmark-outline" unchecked-icon="ion-ios-close-outline" v-model="paradox.inured"/>
									</q-field>
								</div>

								<div class="fieldset">
									<q-field helper="Includes <i>all</i> Paradox rolls not just failed ones.">
										<h6>Number of Previous Paradox Rolls by the Mage in this Scene (+1 dice per roll)</h6>
										<q-input v-model="paradox.previousRolls" type="number" :min="0"/>
									</q-field>
								</div>

								<div class="fieldset">
									<h6>Are There Any Sleeper Witnesses?</h6>
									<q-toggle checked-icon="ion-ios-checkmark-outline" unchecked-icon="ion-ios-close-outline" v-model="paradox.sleepers"/>
								</div>

								<div class="fieldset" v-show="paradox.sleepers">
									<q-field helper="Multiple Sleeper witnesses do not add Paradox dice, but increase the chances of a Paradox occurring. If a few Sleepers witness the magic casting, the Paradox roll gains the 9-Again quality, a large group grants the Paradox roll the 8-Again quality, and a full crowd grants the Paradox roll the rote quality.">
										<h6>How Many Sleeper Witnesses?</h6>
                    <q-radio v-model="paradox.sleeperGroupSize" val="on" label="Just one"/>
										<br>
										<q-radio v-model="paradox.sleeperGroupSize" val="sm" label="A few"/>
										<br>
										<q-radio v-model="paradox.sleeperGroupSize" val="md" label="A large group"/>
										<br>
										<q-radio v-model="paradox.sleeperGroupSize" val="lg" label="A full crowd"/>
									</q-field>
								</div>

								<div class="fieldset">
									<q-field helper="This option is also in the Yantras section.">
										<h6>A Dedicated Magical Tool was Used? (-2 dice)</h6>
										<q-toggle checked-icon="ion-ios-checkmark-outline" unchecked-icon="ion-ios-close-outline" v-model="isDedicatedToolYantraUsed"/>
									</q-field>
								</div>

							</q-card-main>
						</q-card>

					</div>

					<div class="col-xs-12 col-sm-6">

						<q-card>
							<q-card-title>
								Mitigation
							</q-card-title>

							<q-card-separator/>

							<q-card-main>
								<div class="fieldset">
									<q-field helper="Mitigates 1 Paradox Die per Mana spent. Limited by Gnosis.">
										<h6>Additional Mana Spent</h6>
										<q-input v-model="paradox.manaSpent" type="number" :min="0" :max="maxMana"/>
									</q-field>
								</div>
							</q-card-main>
						</q-card>

					</div>
				</div>

				<q-stepper-navigation>
					<q-btn color="secondary" flat @click="stepPrevious()">Back</q-btn>
					<q-btn color="secondary" @click="stepNext()">Next</q-btn>
				</q-stepper-navigation>
			</q-step>

			<q-step name="fifth" title="Summary" icon="ion-ios-analytics-outline" active-icon="ion-ios-analytics-outline" done-icon="ion-ios-analytics-outline">
				<h5>Summary <br class="lt-md">
					<small>Roll for Paradox and to cast the spell</small>
				</h5>

				<div class="row sm-gutter">

					<div class="col-xs-12 col-sm-6">

						<q-card>
							<q-card-title>
								Paradox
							</q-card-title>

							<q-card-separator/>

							<q-card-main>

								<div class="alert-bar alert-bar-info" :class="{ hidden: hasParadox }">
									<q-icon name="ion-ios-checkmark"/>
									This spell does not require a Paradox roll.
								</div>

								<div :class="{ hidden: !hasParadox }">
									<div class="alert-bar alert-bar-warning">
										<q-icon name="ion-alert-circled"/>
										This spell requires a Paradox roll. Roll for Paradox first.
									</div>

									<q-field helper="For Paradox roll results, see Mage: the Awakening 2nd Edition, p. 115">
										<h6>Dice pool</h6>
										<p>
											{{ paradoxDiceSummary }}
											<q-btn small flat @click="showParadoxManaModal()" title="Add">
												<q-icon name="ion-ios-plus"></q-icon>
											</q-btn>
										</p>
									</q-field>

								</div>

							</q-card-main>
						</q-card>

						<q-card v-show="numYantras > 0">
							<q-card-title>
								Yantras
							</q-card-title>

							<q-card-separator/>

							<q-card-main>

								<p v-for="yantra in yantrasSummary">
									<q-field :helper="yantra.desc">
										{{ yantra.name }}
									</q-field>
								</p>

							</q-card-main>
						</q-card>

					</div>

					<div class="col-xs-12 col-sm-6">

						<q-card class="main-card">
							<q-card-title>
								Spell
								<q-chip class="main-card-chip pull-right" v-show="roteOrPraxis === 'rote'">
									<q-icon name="ion-ios-medical"/>
									Rote
								</q-chip>
								<q-chip class="main-card-chip pull-right" v-show="roteOrPraxis === 'praxis'">
									<q-icon name="ion-ios-medical"/>
									Praxis
								</q-chip>
							</q-card-title>

							<q-card-separator/>

							<q-card-main>

								<div class="alert-bar alert-bar-warning" v-show="isDicePoolTooLow">
									<q-icon name="ion-alert-circled"/>
									This spell is too complex and automatically fails.
								</div>

								<div class="alert-bar alert-bar-warning" v-show="isSympatheticYantraMissing">
									<q-icon name="ion-alert-circled"/>
									This spell has Sympathetic Range but does not include any sympathetic Yantras.
									<q-btn flat @click="stepThird()">Back to Yantras</q-btn>
								</div>

								<div v-show="isCastable">

									<div class="alert-bar alert-bar-warning" v-show="isPrimaryFactor( 'potency' ) && subject.isResisted">
										<q-icon name="ion-alert-circled"/>
										This spell is Withstood. Potency {{ totalWithstand + 1 }} is required to affect the target.
									</div>

									<div class="alert-bar alert-bar-info" :class="{ hidden: !spell.isPraxis }">
										<q-icon name="ion-ios-information"/>
										This spell only requires three successes for an exceptional success.
									</div>

									<h6>Arcanum</h6>
									<p>
										{{ caster.arcanaName }}
										<span v-for="i in spell.arcana" class="arcana-dot">&#9679;</span>
									</p>

									<h6>Mana Cost</h6>
									<p>{{ totalMana }}</p>

									<h6>Dice Pool</h6>
									<p>{{ dicePoolSummary }}
										<q-btn small flat @click="showExtraDiceModal()" title="Add">
											<q-icon name="ion-ios-plus"></q-icon>
										</q-btn>
									</p>

									<h6>Successes</h6>
									<p>A single success means the spell takes place. For exceptional successes, see p. 115.</p>

									<div :class="{ hidden: !subject.isResisted }">

										<h6>Withstand</h6>
										<p>{{ totalWithstand }}</p>

										<p v-show="isAdvanced( 'potency')" class="text-muted">
											Advanced Potency has granted -2 to Withstand.
										</p>

									</div>

									<!-- Factors -->
									<q-field helper="As a general rule, each point of Potency grants a one-die bonus or penalty, deals one point of weapon damage, or heals one wound.">
										<h6>Spell Factors</h6>
										<table class="q-table vertical-table">
											<tbody>
											<tr>
												<th>
													Casting Time
													<span v-show="isAdvanced('castingTime')" class="small text-muted"><br>Advanced</span>
												</th>
												<td>
													{{ castingTimeSummary }}
												</td>
											</tr>
											<tr>
												<th>
													Duration
													<span v-show="isAdvanced('duration')" class="small text-muted"><br>Advanced</span>
													<span v-show="isPrimaryFactor('duration')" class="small text-muted"><br>Primary</span>
												</th>
												<td>
													{{ durationSummary }}
												</td>
											</tr>
											<tr>
												<th>
													Potency
													<span v-show="isAdvanced('potency')" class="small text-muted"><br>Advanced</span>
													<span v-show="isPrimaryFactor('potency')" class="small text-muted"><br>Primary</span>
												</th>
												<td>
													{{ potencySummary }}
												</td>
											</tr>
											<tr>
												<th>
													Range <span v-show="isAdvanced('range')" class="small text-muted"><br>Advanced</span>
												</th>
												<td>
													{{ rangeSummary }}
												</td>
											</tr>
											<tr>
												<th>
													Scale <span v-show="isAdvanced('scale')" class="small text-muted"><br>Advanced</span>
												</th>
												<td>
													{{ scaleSummary }}
												</td>
											</tr>
											</tbody>
										</table>
									</q-field>

								</div>

							</q-card-main>
						</q-card>

					</div>
				</div>
				<q-stepper-navigation>
					<q-btn color="secondary" @click="stepFirst()">Restart</q-btn>
					<q-btn color="secondary" flat @click="stepPrevious()">Back</q-btn>
				</q-stepper-navigation>
			</q-step>
		</q-stepper>

		<q-modal ref="settingsModal" position="top" class="settings-modal">
			<q-card>
				<q-card-title>
					Settings
					<q-btn flat color="secondary" slot="right" @click="$refs.settingsModal.close()">
						<q-icon name="ion-ios-close"/>
					</q-btn>
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
								<q-toggle checked-icon="ion-ios-checkmark-outline" unchecked-icon="ion-ios-close-outline" v-model="settings.showAttainments"></q-toggle>
							</q-item-side>
						</q-item>

					</q-list>

				</q-card-main>
			</q-card>
		</q-modal>

		<q-modal ref="yantrasModal" minimised position="top" class="yantra-modal">
			<q-card>
				<q-card-title>
					Yantras
					<q-btn flat color="secondary" slot="right" @click="$refs.yantrasModal.close()">
						<q-icon name="ion-ios-close"/>
					</q-btn>
				</q-card-title>

				<q-card-separator/>

				<q-card-main>

					<q-list link>

						<!-- Locations -->
						<q-list-header>Locations</q-list-header>
						<q-item tag="label" multiline v-for="yantra in locationYantraOptions" :key="yantra.yantraKey" :disabled="!!yantra.disabledWarning" @click="!yantra.disabledWarning ? addYantraFromModal( yantra.yantraKey ) : null">
							<q-item-main>
								<q-item-tile label>
									<b>{{ yantra.name }}</b> (+{{ yantra.bonus }})
									<q-tooltip class="warning" v-if="!!yantra.disabledWarning">
										{{ yantra.disabledWarning }}
									</q-tooltip>
								</q-item-tile>
								<q-item-tile sublabel>{{ yantra.desc }}</q-item-tile>
							</q-item-main>

						</q-item>

						<!-- Actions -->
						<q-list-header>Actions</q-list-header>
						<q-item tag="label" multiline v-for="yantra in actionYantraOptions" :key="yantra.yantraKey" :disabled="!!yantra.disabledWarning" @click="!yantra.disabledWarning ? addYantraFromModal( yantra.yantraKey ) : null">
							<q-item-main>
								<q-item-tile label>
									<b>{{ yantra.name }}</b> (+{{ yantra.bonus }})
									<q-tooltip class="warning" v-if="!!yantra.disabledWarning">
										{{ yantra.disabledWarning }}
									</q-tooltip>
								</q-item-tile>
								<q-item-tile sublabel>{{ yantra.desc }}</q-item-tile>
							</q-item-main>
						</q-item>

						<!-- Tools -->
						<q-list-header>Tools</q-list-header>
						<q-item tag="label" multiline v-for="yantra in toolYantraOptions" :key="yantra.yantraKey" :disabled="!!yantra.disabledWarning" @click="!yantra.disabledWarning ? addYantraFromModal( yantra.yantraKey ) : null">
							<q-item-main>
								<q-item-tile label>
									<b>{{ yantra.name }}</b> (+{{ yantra.bonus }})
									<q-tooltip class="warning" v-if="!!yantra.disabledWarning">
										{{ yantra.disabledWarning }}
									</q-tooltip>
								</q-item-tile>
								<q-item-tile sublabel>{{ yantra.desc }}</q-item-tile>
							</q-item-main>
						</q-item>

					</q-list>

				</q-card-main>
			</q-card>
		</q-modal>

	</q-layout>
</template>

<script>

	import MyYantra from './Yantra.vue'

	import {
		// Quasar components
		QLayout,
		QToolbar, QToolbarTitle,
		QChip,
		QStepper, QStep, QStepperNavigation,
		QCard, QCardTitle, QCardSeparator, QCardMain,
		QList, QListHeader,
		QItem, QItemSide, QItemMain, QItemTile,
		QCollapsible,
		QModal,
		QBtn,
		QTooltip,
		QIcon,
		QField, QSlider, QSelect, QToggle, QRadio, QInput, QCheckbox,
		// Quasar utilities
		Dialog,
		Toast,
		openURL,
		debounce,
		frameDebounce
	} from 'quasar'

	import _ from 'lodash'

	import store from 'store'

	import {
		AddressbarColor
	} from 'quasar'

	AddressbarColor.set( '#155D77' )

	const arcanaNames = [
		'Death',
		'Fate',
		'Forces',
		'Life',
		'Matter',
		'Mind',
		'Prime',
		'Spirit',
		'Space',
		'Time'
	]

	const baseCastingTimes = new Map( [
		[ 9, {
			increment: 1,
			unit: 'minute',
		} ],
		[ 7, {
			increment: 10,
			unit: 'minute',
		} ],
		[ 5, {
			increment: 30,
			unit: 'minute',
		} ],
		[ 3, {
			increment: 1,
			unit: 'hour',
		} ],
		[ 1, {
			increment: 3,
			unit: 'hour',
		} ]
	] )

	const factors = [
		'castingTime',
		'duration',
		'potency',
		'range',
		'scale'
	]

	const gnosisManaLimits = {
		1: 10,
		2: 11,
		3: 12,
		4: 13,
		5: 15,
		6: 20,
		7: 25,
		8: 30,
		9: 50,
		10: 75
	}

	const durations = new Map( [
		[ 's1', {
			time: '1 turn',
			penalty: 0
		} ],
		[ 's2', {
			time: '2 turns',
			penalty: 2
		} ],
		[ 's3', {
			time: '3 turns',
			penalty: 4
		} ],
		[ 's4', {
			time: '5 turns',
			penalty: 6
		} ],
		[ 's5', {
			time: '10 turns',
			penalty: 8
		} ],
		[ 's6', {
			time: '20 turns',
			penalty: 10
		} ],
		[ 's7', {
			time: '30 turns',
			penalty: 12
		} ],
		[ 's8', {
			time: '40 turns',
			penalty: 14
		} ],
		[ 's9', {
			time: '50 turns',
			penalty: 16
		} ],
		[ 's10', {
			time: '60 turns',
			penalty: 18
		} ],
		[ 's11', {
			time: '70 turns',
			penalty: 20
		} ],
		[ 'a1', {
			time: '1 scene/hour',
			penalty: 0
		} ],
		[ 'a2', {
			time: '1 day',
			penalty: 2
		} ],
		[ 'a3', {
			time: '1 week',
			penalty: 4
		} ],
		[ 'a4', {
			time: '1 month',
			penalty: 6
		} ],
		[ 'a5', {
			time: '1 year',
			penalty: 8
		} ],
		[ 'a6', {
			time: 'Indefinite (requires +1 Reach & +1 Mana)',
			penalty: 10
		} ] ] )

	const scales = new Map( [
		[ 's1', {
			number: 1,
			size: '5',
			area: 'Arm’s reach from a central point',
			penalty: 0
		} ],
		[ 's2', {
			number: 2,
			size: '6',
			area: 'A small room',
			penalty: 2
		} ],
		[ 's3', {
			number: 4,
			size: '7',
			area: 'A large room',
			penalty: 4
		} ],
		[ 's4', {
			number: 8,
			size: '8',
			area: 'Several rooms, or a single floor of a house',
			penalty: 6
		} ],
		[ 's5', {
			number: 16,
			size: '9',
			area: 'A ballroom or small house',
			penalty: 8
		} ],
		[ 's6', {
			number: 32,
			size: '10',
			area: 'A ballroom or small house',
			penalty: 10
		} ],
		[ 's7', {
			number: 64,
			size: '11',
			area: 'A ballroom or small house',
			penalty: 12
		} ],
		[ 's8', {
			number: 128,
			size: '12',
			area: 'A ballroom or small house',
			penalty: 14
		} ],
		[ 's9', {
			number: 256,
			size: '13',
			area: 'A ballroom or small house',
			penalty: 16
		} ],
		[ 'a1', {
			number: 'Five',
			size: '5',
			area: 'A large house or building',
			penalty: 0
		} ],
		[ 'a2', {
			number: '10',
			size: '10',
			area: 'A small warehouse or parking lot',
			penalty: 2
		} ],
		[ 'a3', {
			number: '20',
			size: '15',
			area: 'A large warehouse or supermarket',
			penalty: 4
		} ],
		[ 'a4', {
			number: '40',
			size: '20',
			area: 'A small factory, or a shopping mall',
			penalty: 6
		} ],
		[ 'a5', {
			number: '80',
			size: '25',
			area: 'A large factory, or a city block',
			penalty: 8
		} ],
		[ 'a6', {
			number: '160',
			size: '30',
			area: 'A campus, or a small neighborhood',
			penalty: 10
		} ]
		,
		[ 'a7', {
			number: '320',
			size: '35',
			area: 'A campus, or a small neighborhood',
			penalty: 12
		} ]
		,
		[ 'a8', {
			number: '640',
			size: '40',
			area: 'A campus, or a small neighborhood',
			penalty: 14
		} ]
		,
		[ 'a9', {
			number: '1280',
			size: '45',
			area: 'A campus, or a small neighborhood',
			penalty: 16
		} ]
	] )

	/**
	 * Expanded and modified by other data in the yantras computed property
	 */
	const yantrasBaseData = new Map( [

		// locations.
		[ 'l1', {
			name: 'Demesne',
			desc: 'A prepared ritual space with a soul stone',
			bonus: 2,
			unique: true
		} ],
		[ 'l2', {
			name: 'Location',
			desc: 'A place and time symbolically linked to the spell.',
			bonus: 1,
			unique: true
		} ],
		[ 'l3', {
			name: 'Supernal Verge',
			desc: 'A place where the Supernal touches the Fallen World.',
			bonus: 2,
			unique: true
		} ],

		// actions
		[ 'a1', {
			name: 'Rote Skill Mudra',
			desc: 'Uses skill dots as a bonus. The character must be free to make whatever mnemonic gestures are used to recall the Rote.',
			bonus: 0,
			unique: true
		} ],
		[ 'a2', {
			name: 'Concentration',
			desc: 'Duration must be greater than a turn. If the mage is hurt or takes a non-reflexive action while the spell is active, it ends immediately.',
			bonus: 2,
			unique: true
		} ],
		[ 'a3', {
			name: 'Mantra (High Speech)',
			desc: 'Must be spoken aloud. Cannot be used reflexively.',
			bonus: 2,
			unique: true
		} ],
		[ 'a4', {
			name: 'Runes',
			desc: 'The subject is marked with runes. Ritual casting only. If anything damages or disrupts the runes while the spell is active, it ends immediately.',
			bonus: 2,
			unique: true
		} ],

		// tools
		[ 't1', {
			name: 'Dedicated Tool',
			desc: 'An item that synchronizes with her Nimbus and that feeds in to her understanding of magic. Reduces Paradox by 2 dice.',
			bonus: 0,
			unique: true
		} ],
		[ 't2', {
			name: 'Path Tool',
			desc: 'Tools which align closely to her Path. See p.121 for examples.',
			bonus: 1,
			unique: false
		} ],
		[ 't3', {
			name: 'Order Tool',
			desc: 'Tools which draw upon an Order’s symbols rather than those of the Supernal world directly, focusing magic in a way that matches their teachings.',
			bonus: 1,
			unique: false
		} ],
		[ 't4', {
			name: 'Material Sympathy',
			desc: 'An item sympathetically linked to the subject <i>as they are now</i>. At least one sympathetic tool is required for sympathetic casting. Does not grant a bonus when used with Sympathetic Range or Temporal Sympathy Attainments.',
			bonus: 2,
			unique: false
		} ],
		[ 't5', {
			name: 'Representational Sympathy',
			desc: 'An item sympathetically linked to the subject <i>as they were previously</i>. At least one sympathetic tool is required for sympathetic casting. Does not grant a bonus when used with Sympathetic Range or Temporal Sympathy Attainments.',
			bonus: 1,
			unique: false
		} ],
		[ 't6', {
			name: 'Symbolic Sympathy',
			desc: 'An indirect representation of the subject. At least one sympathetic tool is required for sympathetic casting.',
			bonus: 0,
			unique: false
		} ],
		[ 't7', {
			name: 'Sacrament',
			desc: 'An object symbolic of the spell that the mage destroys during casting.',
			bonus: 1,
			unique: false
		} ],
		[ 't8', {
			name: 'Rare Sacrament',
			desc: 'A sacrament which requires significant effort to acquire.',
			bonus: 2,
			unique: false
		} ],
		[ 't9', {
			name: 'Otherworldly Sacrament',
			desc: 'A sacrament from somewhere other than the material realm.',
			bonus: 3,
			unique: false
		} ],
		[ 't10', {
			name: 'Persona',
			desc: 'A persona Yantra keys in to the mage’s Shadow Name and Cabal Theme Merits.',
			bonus: [ 1, 4 ],
			unique: true
		} ]
	] )

	var getInitialData = function ()
	{
		return {
			currentStep: 'first',
			caster: {
				gnosis: 1,
				arcana: 1,
				arcanaName: arcanaNames[ 0 ],
				isHighestArcana: true,
				isRulingArcana: true,
				activeSpells: 0
			},
			spell: {
				arcana: 1,
				isPraxis: false,
				primaryFactor: 'potency',
				factors: {
					castingTime: 's1',
					potency: 's1',
					range: 's1',
					duration: 's1',
					scale: 's1',
				},
				isRote: false,
				roteSkill: 1,
				bonusDice: 0,
				spendWillpower: false,
				extraReach: 0,
				yantras: [],
				yantraAlsoDedicatedTool: null,
				attainments: {
					conditionalDuration: false,
					everywhere: false,
					permanence: false,
					sympatheticRange: false,
					temporalSympathy: false,
					timeInABottle: false
				}
			},
			subject: {
				isResisted: false,
				withstand: 1,
				numWithstands: 1
			},
			paradox: {
				inured: false,
				previousRolls: 0,
				sleepers: false,
				sleeperGroupSize: 'on',
				manaSpent: 0
			},
			settings: {
				showAttainments: true
			}
		}
	}

	// Toast Defaults
	Toast.setDefaults( {
		icon: 'ion-ios-lightbulb',
		timeout: 3500
	} )

	export default {
		name: 'spell-helper',
		components: {
			MyYantra,
			QLayout,
			QToolbar, QToolbarTitle,
			QChip,
			QStepper, QStep, QStepperNavigation,
			QCard, QCardTitle, QCardSeparator, QCardMain,
			QList, QListHeader,
			QItem, QItemSide, QItemMain, QItemTile,
			QCollapsible,
			QModal,
			QBtn,
			QTooltip,
			QIcon,
			QField, QSlider, QSelect, QToggle, QRadio, QInput, QCheckbox,
		},
		data: function ()
		{
			return getInitialData()
		},
		computed: {
			arcanaNameOptions()
			{
				let options = []

				for ( let arcanaName of arcanaNames )
				{
					options.push( {
						label: arcanaName,
						value: arcanaName
					} )
				}

				return options
			},
			maxCasterArcana()
			{
				let arcana

				if ( this.caster.isHighestArcana )
				{
					if ( this.caster.gnosis >= 5 )
					{
						arcana = 5
					}
					else if ( this.caster.gnosis >= 3 )
					{
						arcana = 4
					}
					else
					{
						arcana = 3
					}
				}
				else
				{
					if ( this.caster.gnosis >= 6 )
					{
						arcana = 5
					}
					else if ( this.caster.gnosis >= 4 )
					{
						arcana = 4
					}
					else
					{
						arcana = 3
					}
				}

				return arcana
			},
			isCasterArcanaTooHigh()
			{
				return this.caster.arcana > this.maxCasterArcana
			},
			isSpellArcanaTooHigh()
			{
				return this.caster.arcana < this.spell.arcana
			},
			freeReach()
			{
				let arcana

				if ( this.spell.isRote )
				{
					arcana = 5
				}
				else
				{
					arcana = this.caster.arcana
				}

				return arcana - this.spell.arcana + 1
			},
			usedReach()
			{
				let reach = 0

				if ( this.caster.activeSpells >= this.caster.gnosis )
				{
					reach += this.caster.activeSpells - this.caster.gnosis + 1
				}

				// check factors (advanced factor keys begin with "a")
				for ( let factor of factors )
				{
					if ( this.spell.factors[ factor ][ 0 ] === 'a' )
					{
						reach++
					}
				}

				// indefinite duration costs 1 reach
				if ( !this.isPrimaryFactor( 'duration' ) && this.spell.factors.duration === 'a6' )
				{
					reach++
				}

				// spell-specific extra reach
				reach += this.spell.extraReach

				if ( this.spell.attainments.permanence )
				{
					reach--
				}

				if ( this.spell.attainments.timeInABottle )
				{
					reach--
				}

				if ( this.spell.attainments.everywhere )
				{
					reach--
				}

				return reach
			},
			roteOrPraxis()
			{
				if ( this.spell.isRote )
				{
					return 'rote'
				}
				else if ( this.spell.isPraxis )
				{
					return 'praxis'
				}
				else
				{
					return null
				}
			},
			baseParadoxDice()
			{
				return Math.ceil( this.caster.gnosis / 2 )
			},
			paradoxDice()
			{
				let pool,
				    mustRoll

				pool = ( this.freeReach - this.usedReach ) * -1

				mustRoll = pool > 0

				// gnosis multiplies paradox from additional reach
				pool *= this.baseParadoxDice

				if ( this.paradox.inured )
				{
					pool += 2
					mustRoll = true
				}

				if ( this.paradox.previousRolls > 0 )
				{
					pool += this.paradox.previousRolls
					mustRoll = true
				}

				if ( this.paradox.sleepers )
				{
					pool++
					mustRoll = true
				}

				if ( this.isDedicatedToolYantraUsed )
				{
					pool -= 2
				}

				pool -= this.paradox.manaSpent

				if ( pool <= 0 && mustRoll )
				{
					return '[chance die]'
				}

				return pool
			},
			hasParadox()
			{
				return this.usedReach > this.freeReach ||
					this.paradox.inured ||
					this.paradox.previousRolls > 0 ||
					this.paradox.sleepers;
			},
			durationPenalty()
			{
				let penalty = durations.get( this.spell.factors.duration ).penalty

				if ( this.isPrimaryFactor( 'duration' ) )
				{
					penalty -= ( this.caster.arcana - 1 ) * 2
				}

				if ( penalty <= 0 )
				{
					penalty = 0
				}

				return penalty
			},
			potencyValue()
			{
				return this.spell.factors.potency.substr( 1 )
			},
			potencyPenalty()
			{
				let penalty = ( this.potencyValue - 1 ) * 2

				if ( this.isPrimaryFactor( 'potency' ) )
				{
					penalty -= ( this.caster.arcana - 1 ) * 2
				}

				if ( penalty <= 0 )
				{
					penalty = 0
				}

				return penalty
			},
			dicePool()
			{
				// base pool
				let pool = this.caster.gnosis + this.caster.arcana

				pool += this.spell.bonusDice

				if ( this.spell.spendWillpower )
				{
					pool += 3
				}

				// casting time
				if ( !this.isAdvanced( 'castingTime' ) )
				{
					pool += this.spell.factors.castingTime[ 1 ] - 1
				}

				// potency
				pool -= this.potencyPenalty

				// duration
				pool -= this.durationPenalty

				// scale
				pool -= scales.get( this.spell.factors.scale ).penalty

				// yantras
				this.spell.yantras.forEach( yantra =>
				{
					pool += yantra.bonus
				} )

				return pool
			},
			isDicePoolTooLow()
			{
				return this.dicePool < -5
			},
			spellFactorOptions()
			{
				return [ {
					label: 'Duration',
					value: 'duration'
				},
					{
						label: 'Potency',
						value: 'potency'
					} ]
			},
			primaryFactor()
			{
				return this.spell.primaryFactor
			},
			primaryFactorLabel()
			{
				return _.capitalize( this.spell.primaryFactor )
			},
			totalWithstand()
			{
				if ( !this.subject.isResisted )
				{
					return 0
				}

				let withstand = this.subject.withstand + this.subject.numWithstands - 1

				if ( this.spell.factors.potency === 'a1' )
				{
					withstand -= 2
				}

				return withstand
			},
			yantras()
			{
				let expandedYantras = new Map()

				for ( let [ key, yantraBaseData ] of yantrasBaseData )
				{
					// bonus can contain a single number or a range. Arrayify.
					let bonuses
					if ( Array.isArray( yantraBaseData.bonus ) )
					{
						bonuses = _.range( yantraBaseData.bonus[ 0 ], yantraBaseData.bonus[ 1 ] + 1 )
					}
					else
					{
						bonuses = [ yantraBaseData.bonus ]
					}

					bonuses.forEach( bonus =>
					{
						let expandedYantra = _.clone( yantraBaseData )

						/*
						 * Bonus
						 */

						// rote skill mudra: bonus = skill dots
						if ( key === 'a1' && this.spell.isRote )
						{
							bonus = this.spell.roteSkill
						}

						// sympathetic yantras don't give a bonus to sympathetic or temporal spells
						if ( [ 't4', 't5' ].includes( key ) && ( this.spell.attainments.sympatheticRange || this.spell.attainments.temporalSympathy ) )
						{
							bonus = 0
						}

						expandedYantra.yantraKey = Array.isArray( yantraBaseData.bonus ) ? key + '_' + bonus : key // key is a reserved property in Vue so we use "yantraKey"
						expandedYantra.bonus = bonus
						expandedYantra.label = `${yantraBaseData.name} (+${bonus} ${bonus === 1 ? 'die' : 'dice'})`
						expandedYantra.isDedicatedTool = false
						expandedYantras.set( expandedYantra.yantraKey, expandedYantra )
					} )
				}

				return expandedYantras
			},
			maxYantras()
			{
				return Math.ceil( this.caster.gnosis / 2 ) + 1
			},
			numYantras()
			{
				return this.spell.yantras.length
			},
			isConcentrationMantraAllowed()
			{
				return this.isPrimaryFactor( 'duration' ) || this.spell.factors.duration !== 's1'
			},
			isDedicatedToolYantraUsed:
				{
					cache: false,
					// getter
					get()
					{
						if ( this.hasYantra( 't1' ) )
						{
							return true
						}

						return _.some( this.spell.yantras, [ 'isDedicatedTool', true ] )
					},
					// setter (bool)
					set( isUsed )
					{
						if ( isUsed )
						{
							if ( this.isDedicatedToolYantraUsed )
							{
								debounce(
									() =>
									{
										Toast.create( 'A Dedicated Tool is already selected' )
									}
								)()
							}

							this.spell.yantras.push( this.yantras.get( 't1' ) )
						}
						else
						{
							this.deleteYantra( 't1' )

							for ( key of this.spell.yantras )
							{
								this.spell.yantras[ key ].isDedicatedTool = false
							}

							this.$forceUpdate() // cache: false and this.$forceUpdate() means the getter shows updated value
						}
					}
				},
			totalMana()
			{
				let mana = 0;

				if ( !this.caster.isRulingArcana && !this.spell.isRote && !this.spell.isPraxis )
				{
					mana++
				}

				if ( this.spell.factors.duration === 'a6' )
				{
					mana++
				}

				if ( this.spell.attainments.permanence )
				{
					mana++
				}

				if ( this.spell.attainments.timeInABottle )
				{
					mana++
				}

				if ( this.spell.attainments.sympatheticRange )
				{
					mana++
				}

				if ( this.spell.attainments.temporalSympathy )
				{
					mana++
				}

				if ( this.spell.attainments.everywhere )
				{
					mana++
				}

				mana += this.paradox.manaSpent

				return mana
			},
			maxMana()
			{
				return gnosisManaLimits[ this.caster.gnosis ]
			},
			// use this to watch attainments changing
			attainmentsByName()
			{
				let attainments = []

				_.each( this.spell.attainments, ( value, key ) =>
				{
					if ( value )
					{
						attainments.push( key )
					}
				} )

				return attainments
			},
			isDicePoolTooLow()
			{
				return this.dicePool < -5
			},
			isSympatheticYantraMissing()
			{
				return ( this.spell.attainments.sympatheticRange || this.spell.attainments.temporalSympathy ) &&
					!this.hasYantra( 't3' ) && !this.hasYantra( 't4' ) && !this.hasYantra( 't5' )
			},
			isCastable()
			{
				return !this.isDicePoolTooLow && !this.isSympatheticYantraMissing
			},
			baseCastingTime()
			{
				for ( let [ key, value ] of baseCastingTimes )
				{
					if ( this.caster.gnosis >= key )
					{
						return value
					}
				}
			},
			standardCastingTimeOptions()
			{
				let options = [],
				    i       = 0

				while ( i++ <= 5 )
				{
					let increment = this.baseCastingTime.increment * i,
					    unit      = this.baseCastingTime.unit + ( increment !== 1 ? 's' : '' ),
					    bonus     = i - 1

					options.push( {
						key: 's' + i,
						label: `${ increment } ${ unit } (+${ bonus } dice)`
					} )
				}

				return options
			},
			standardPotencyOptions()
			{
				let options = [],
				    i       = 0

				while ( i++ < 11 )
				{
					let penalty = ( i - 1 ) * 2

					if ( this.isPrimaryFactor( 'potency' ) )
					{
						penalty -= ( this.caster.arcana - 1 ) * 2
					}

					if ( penalty < 0 )
					{
						penalty = 0
					}

					options.push( {
						key: 's' + i,
						value: i,
						label: `${i} (-${penalty} dice)`
					} )
				}

				return options
			},
			advancedPotencyOptions()
			{
				let options = []
				let i = 0
				while ( i++ < 11 )
				{
					let penalty = ( i - 1 ) * 2

					if ( this.isPrimaryFactor( 'potency' ) )
					{
						penalty -= ( this.caster.arcana - 1 ) * 2
					}

					if ( penalty < 0 )
					{
						penalty = 0
					}

					options.push( {
						key: 'a' + i,
						value: i,
						label: `${i} (-${penalty} dice)`
					} )
				}

				return options
			},
			standardDurationOptions()
			{
				let options = []

				for ( let [ key, duration ] of durations )
				{
					if ( key[ 0 ] === 's' )
					{
						let penalty = duration.penalty

						if ( this.isPrimaryFactor( 'duration' ) )
						{
							penalty -= ( this.caster.arcana - 1 ) * 2
						}

						if ( penalty < 0 )
						{
							penalty = 0
						}

						options.push( {
							key: key,
							successes: key[ 1 ],
							time: duration.time,
							label: `${duration.time} (-${penalty} dice)`
						} )
					}
				}

				return options
			},
			advancedDurationOptions()
			{
				let options = []

				for ( let [ key, duration ] of durations )
				{
					if ( key[ 0 ] === 'a' )
					{
						let penalty = duration.penalty

						if ( this.isPrimaryFactor( 'duration' ) )
						{
							penalty -= ( this.caster.arcana - 1 ) * 2
						}

						if ( penalty < 0 )
						{
							penalty = 0
						}

						options.push( {
							key: key,
							successes: key[ 1 ],
							time: duration.time,
							label: `${duration.time} (-${penalty} dice)`
						} )
					}
				}

				return options
			},
			standardScaleOptions()
			{
				let options = []

				for ( let [ key, scale ] of scales )
				{
					if ( key[ 0 ] === 's' )
					{
						let s = _.clone( scale )
						s.key = key
						s.value = key[ 1 ]
						s.penalty = scale.penalty * -1
						options.push( s )
					}
				}

				return options
			},
			advancedScaleOptions()
			{
				let options = []

				for ( let [ key, scale ] of scales )
				{
					if ( key[ 0 ] === 'a' )
					{
						let s = _.clone( scale )
						s.key = key
						s.value = key[ 1 ]
						s.penalty = scale.penalty * -1
						options.push( s )
					}
				}

				return options
			},
			locationYantraOptions()
			{
				return this.getYantraOptions( 'l' )
			},
			actionYantraOptions()
			{
				return this.getYantraOptions( 'a' )
			},
			toolYantraOptions()
			{
				return this.getYantraOptions( 't' )
			},
			paradoxDiceSummary()
			{
				let summary

				if ( this.paradoxDice === '[chance die]' )
				{
					summary = 'A chance die'
				}
				else if ( this.paradoxDice === 1 )
				{
					summary = this.paradoxDice + ' die'
				}
				else
				{
					summary = this.paradoxDice + ' dice'
				}

				if ( this.paradox.sleepers )
				{
					if ( this.paradox.sleeperGroupSize === 'sm' )
					{
						summary += ' (with the 9-again quality)'
					}
					else if ( this.paradox.sleeperGroupSize === 'md' )
					{
						summary += ' (with the 8-again quality)'
					}
					else if ( this.paradox.sleeperGroupSize === 'lg' )
					{
						summary += ' (with the rote quality)'
					}
				}

				return summary
			},
			dicePoolSummary()
			{
				if ( this.dicePool < 1 )
				{
					return 'A chance die'
				}
				else if ( this.dicePool == 1 )
				{
					return `${ this.dicePool } die`
				}
				else
				{
					return `${ this.dicePool } dice`
				}
			},
			castingTimeSummary()
			{
				// standard
				if ( this.spell.factors.castingTime[ 0 ] === 's' )
				{
					let increment = this.baseCastingTime.increment * this.spell.factors.castingTime[ 1 ],
					    unit      = this.baseCastingTime.unit + ( increment !== 1 ? 's' : '' )

					return increment + ' ' + unit
				}
				// advanced
				else
				{
					let turns = this.numYantras <= 1 ? 1 : this.numYantras;

					if ( this.hasYantra( 'a3' ) )
					{
						turns = turns == 1 ? 2 : turns
					}

					return `${ turns } turn${ turns !== 1 ? 's' : '' }`
				}
			},
			durationSummary()
			{
				return durations.get( this.spell.factors.duration ).time
			},
			potencySummary()
			{
				return this.potencyValue
			},
			rangeSummary()
			{
				if ( this.spell.factors.range === 's1' )
				{
					return 'Touch/aimed'
				}
				else if ( this.spell.attainments.sympatheticRange || this.spell.attainments.temporalSympathy )
				{
					let range = []

					if ( this.spell.attainments.sympatheticRange )
					{
						range.push( 'Sympathetic' )
					}

					if ( this.spell.attainments.temporalSympathy )
					{
						range.push( 'Temporal Sympathetic' )
					}

					return range.join( ' and ' )
				}
				else
				{
					return 'Sensory'
				}
			},
			scaleSummary()
			{
				let scale = scales.get( this.spell.factors.scale )

				return `${scale.number} subjects of up to size ${scale.size} or subjects ${ scale.area.toLowerCase() }`
			},
			yantrasSummary()
			{
				let yantrasNames = []

				for ( let yantra of this.spell.yantras )
				{
					yantrasNames.push( this.yantras.get( yantra.yantraKey ) )
				}

				return yantrasNames
			},
			factorCastingTime()
			{
				return this.spell.factors.castingTime
			},
			factorDuration()
			{
				return this.spell.factors.duration
			},
			factorRange()
			{
				return this.spell.factors.range
			},
			factorScale()
			{
				return this.spell.factors.scale
			}
		},
		watch: {
			/*currentStep()
			{
			},*/
			freeReach()
			{
				this.animateChip( 'footer-reach-chip' )
			},
			usedReach()
			{
				this.animateChip( 'footer-reach-chip' )
			},
			dicePool()
			{
				this.animateChip( 'footer-dice-pool-chip' )
			},
			totalMana()
			{
				this.animateChip( 'footer-mana-chip' )
			},
			roteOrPraxis()
			{
				if ( this.spell.isRote )
				{
					this.addYantra( 'a1' )
				}
				else
				{
					this.deleteYantra( 'a1' )
				}
			},
			paradoxDice()
			{
				this.animateChip( 'footer-paradox-chip' )
			},
			isCasterArcanaTooHigh() // watch computed so we don't have to use inefficient deep watcher
			{
				if ( this.caster.arcana > this.maxCasterArcana )
				{
					debounce(
						() =>
						{
							this.caster.arcana = this.maxCasterArcana
							Toast.create( `At Gnosis ${ this.caster.gnosis },
								Caster's ${ !this.caster.isHighestArcana ? 'non-' : '' } highest Arcana
								cannot exceed ${ this.maxCasterArcana }` )
						},
						500
					)()
				}

			},
			isSpellArcanaTooHigh()
			{
				if ( this.spell.arcana > this.caster.arcana )
				{
					debounce(
						() =>
						{
							this.spell.arcana = this.caster.arcana
							Toast.create( `Spell's Arcana cannot exceed Caster's Arcana of ${this.caster.arcana}` )
						},
						500
					)()
				}
			},
			primaryFactor()
			{
				this.checkPotencyAgainstWithstand()
			},
			isConcentrationMantraAllowed()
			{
				if ( !this.isConcentrationMantraAllowed && this.hasYantra( 'a2' ) )
				{
					this.deleteYantra( 'a2' )
				}
			},
			// disable potency checkboxes below Withstand level
			totalWithstand()
			{
				this.checkPotencyAgainstWithstand()
			},
			attainmentsByName()
			{
				this.checkAttainments()
			},
			factorCastingTime()
			{
				this.checkAttainments()
			},
			factorDuration()
			{
				this.checkAttainments()
			},
			factorRange()
			{
				this.checkAttainments()
			},
			factorScale()
			{
				this.checkAttainments()
			},
			numYantras()
			{
				if ( this.numYantras > this.maxYantras )
				{
					debounce(
						() =>
						{
							this.spell.yantras.pop()
							Toast.create( 'Gnosis ' + this.caster.gnosis + ' only allows ' + this.maxYantras + ' yantras' )
						}
					)()
				}
			},
			settings:
				{
					handler()
					{
						store.set( 'settings', this.settings )
					},
					deep: true
				}
		},
		methods: {
			open( url )
			{
				openURL( url )
			},
			isAdvanced( factorName )
			{
				return this.spell.factors[ factorName ][ 0 ] === 'a'
			},
			isPrimaryFactor( factorName )
			{
				return this.spell.primaryFactor === factorName
			},
			/**
			 * Called when Caster's Arcana, Potency, Primary Factor or Withstand changed
			 */
			checkPotencyAgainstWithstand()
			{
				let extraPotency = this.spell.factors.potency[ 1 ] - 1,
				    prefix       = this.spell.factors.potency[ 0 ], // 's' or 'a'
				    minPotency   = this.subject.isResisted ? this.totalWithstand + 1 : 0,
				    isStandard   = prefix !== 'a'

				// Advanced Potency reduces Withstand by 2
				if ( !isStandard )
				{
					minPotency -= 2
				}

				// is Potency too low?
				if ( extraPotency < minPotency )
				{
					debounce(
						() =>
						{
							this.spell.factors.potency = prefix + minPotency
							Toast.create( `Potency spell factor (${ prefix === 's' ? 'Standard' : 'Advanced' }) automatically increased to ${ minPotency }` )
						}
					)()
				}
			},
			checkAttainments()
			{
				// time in a bottle requires advanced
				if ( this.spell.attainments.timeInABottle && !this.isAdvanced( 'castingTime' ) )
				{
					debounce(
						() =>
						{
							this.spell.attainments.timeInABottle = false
							Toast.create( 'Time in a Bottle attainment requires advanced Casting Time. Deselecting attainment.' )
						}
					)()
				}

				// permanence requires advanced
				if ( this.spell.attainments.permanence && ( this.caster.arcanaName !== 'Matter' || !this.isAdvanced( 'duration' ) ) )
				{
					debounce(
						() =>
						{
							this.spell.attainments.permanence = false
							Toast.create( 'Permanence attainment requires advanced Duration. Deselecting attainment.' )
						}
					)()
				}

				// everywhere requires advanced
				if ( this.spell.attainments.everywhere && !this.isAdvanced( 'scale' ) )
				{
					debounce(
						() =>
						{
							this.spell.attainments.everywhere = false
							Toast.create( 'Everywhere attainment requires advanced Scale. Deselecting attainment.' )
						}
					)()
				}

				// sympathy requires advanced
				if ( this.spell.attainments.sympatheticRange && !this.isAdvanced( 'range' ) )
				{
					debounce(
						() =>
						{
							this.spell.attainments.sympatheticRange = false
							Toast.create( 'Sympathetic Range attainment requires advanced Range. Deselecting attainment.' )
						}
					)()
				}

				// temporal sympathy requires advanced
				if ( this.spell.attainments.temporalSympathy && !this.isAdvanced( 'range' ) )
				{
					debounce(
						() =>
						{
							this.spell.attainments.temporalSympathy = false
							Toast.create( 'Temporal Sympathy attainment requires advanced Range. Deselecting attainment.' )
						}
					)()
				}
			},
			isUniqueYantraUsed( key )
			{
				let yantra = this.yantras.get( key )

				// ignore non-uniques
				if ( !yantra.unique )
				{
					return false
				}

				// this version is used?
				if ( this.hasYantra( key ) )
				{
					return true
				}

				// there are multiple version of this yantra - check for others
				if ( key.indexOf( '_' ) !== -1 )
				{
					let baseKey = key.split( '_' )[ 0 ];
					if ( this.spell.yantras.some( yantra => yantra.yantraKey.indexOf( baseKey ) === 0 ) )
					{
						return true
					}
				}

				return false
			},
			getYantraOptions( prefix )
			{
				let options = []

				for ( let [ key, yantra ] of this.yantras )
				{
					if ( key[ 0 ] === prefix ) // 'l', 'a', or 't'
					{
						// disabled?
						let disabledWarning

						if ( key === 'a1' && !this.spell.isRote )
						{
							disabledWarning = 'Only available when casting Rotes.'
						}

						if ( key === 'a2' && !this.isConcentrationMantraAllowed )
						{
							disabledWarning = 'Duration must be more than 1 turn to use concentration.'
						}

						if ( this.isUniqueYantraUsed( key ) )
						{
							disabledWarning = 'Only one of this Yantra may be used.'
						}

						if ( key === 't1' && this.isDedicatedToolYantraUsed )
						{
							disabledWarning = 'A Dedicated Tool is already being used.'
						}

						yantra.disabledWarning = disabledWarning

						// add to options
						options.push( yantra )
					}
				}

				return options
			},
			hasYantra( key )
			{
				return _.some( this.spell.yantras, [ 'yantraKey', key ] ) // `_.matchesProperty` iteratee shorthand.
			},
			addYantra( key )
			{
				let yantra = this.yantras.get( key )

				// check uniques
				if ( this.isUniqueYantraUsed( key ) )
				{
					debounce(
						() =>
						{
							Toast.create( `The ${ yantra.name } Yantra can only be selected once` )
						}
					)()
					return
				}

				this.spell.yantras.push( yantra )
			},
			deleteYantra( key )
			{
				let index = _.findIndex( this.spell.yantras, yantra => yantra.yantraKey === key )
				this.spell.yantras.splice( index, 1 )
			}
			,
			addYantraFromModal( key )
			{
				this.addYantra( key )

				this.$refs.yantrasModal.close()
			}
			,
			updateYantraIsDedicatedTool( key, newValue )
			{
				let index = _.findIndex( this.spell.yantras, yantra => yantra.yantraKey === key )
				this.spell.yantras[ index ].isDedicatedTool = newValue
			}
			,
			stepNext()
			{
				this.$refs.stepper.next()
				this.scrollUp()
			}
			,
			stepPrevious()
			{
				this.$refs.stepper.previous()
				this.scrollUp()
			}
			,
			stepFirst()
			{
				this.$refs.stepper.goToStep( 'first' )
				this.scrollUp()
			}
			,
			stepThird()
			{
				this.$refs.stepper.goToStep( 'third' )
				this.scrollUp()
			}
			,
			scrollUp()
			{
				window.scrollTo( 0, 0 );
			}
			,
			/*disableStepButtonFirst()
			{
				document.getElementById( 'step-button-first' ).setAttribute( 'disabled', true )
			},
			enableStepButtonFirst()
			{
				document.getElementById( 'step-button-first' ).removeAttribute( 'disabled' )
			},*/
			showExtraDiceModal()
			{
				Dialog.create( {
					title: 'Add Spellcasting Dice',
					message: 'Change the bonus dice added in the Spell Overview section.',
					position: 'top',
					form: {
						bonusDice: {
							type: 'number',
							label: 'Bonus Dice',
							model: this.spell.bonusDice,
							min: 0,
							withLabel: true
						},
						spendWillpower: {
							type: 'toggle',
							label: 'Spend Willpower',
							items: [ {
								label: 'Spend Willpower',
								value: 'spend'
							} ],
							model: this.spell.spendWillpower,
							withLabel: true
						}
					},
					buttons: [
						'Cancel',
						{
							label: 'Save',
							handler: ( data ) =>
							{
								this.spell.bonusDice = data.bonusDice;
								this.spell.spendWillpower = data.spendWillpower;
							}
						}
					]
				} )
			}
			,
			showParadoxManaModal()
			{
				Dialog.create( {
					title: 'Mitigate Paradox Dice',
					message: 'Spend Mana. Each point of Mana removes one dice.',
					position: 'top',
					form: {
						manaSpent: {
							type: 'number',
							label: 'Additional Mana Spent',
							model: this.paradox.manaSpent,
							min: 0,
							withLabel: true
						}
					},
					buttons: [
						'Cancel',
						{
							label: 'Save',
							handler: ( data ) =>
							{
								this.paradox.manaSpent = data.manaSpent;
							}
						}
					]
				} )
			}
			,
			animateChip( id )
			{
				let chip = document.getElementById( id )

				// add animation class
				chip.classList.add( 'animate' )

				// remove again
				for ( let eventName of [ 'webkitAnimationEnd', 'mozAnimationEnd', 'MSAnimationEnd', 'oanimationend', 'animationend' ] )
				{
					chip.addEventListener( eventName,
						() =>
						{
							chip.classList.remove( 'animate' )
						},
						{ once: true } )
				}
			}
			,
			resetData()
			{
				Object.assign( this.$data, getInitialData() )
			}
		},
		mounted()
		{
			// load persisted settings?
			let storedSettings = store.get( 'settings' )
			if ( storedSettings &&
				'showAttainments' in storedSettings )
			{
				this.settings = storedSettings
			}
		}
	}
</script>

<style lang="stylus">
	@import '~variables'

	.layout-page
		margin-bottom 40px

	.q-toolbar-title a
		color white
		&:hover
			color $secondary

	.toolbar-spacer
		padding 0 1rem

	.text-muted
		color $faded

	.q-stepper-header
		box-shadow unset
		padding-top 0

	.q-stepper-horizontal .q-stepper-step-inner
		padding-top 0
		h5
			margin-top 2px

	.q-stepper-tab
		padding: 32px 16px 28px !important

	.q-stepper-dot
		width 36px
		height 36px
		i.q-icon
			font-size 22px

	.q-stepper-nav
		position fixed
		bottom 0
		right 16px
		z-index 2100

	.sm-gutter > div > .q-card
		margin 0
		& + .q-card
			margin-top 16px

	h5 small
		display inline-block
		font-size 1rem
		font-weight 400
		line-height 110%
		color #777

	h6
		font-size 18px

	p
		margin-bottom 8px
		& > small
			font-size 14px
			line-height 11px

	span.arcana-dot
		color black !important
		font-size 90%

	p .q-btn.inline
		margin-top -3px
		padding 0 8px

	.q-card-primary
		background-color $primary
		color $white
		.q-btn
			color $white !important
			padding 0 8px

	.q-card-title
		font-size 18px
		text-transform uppercase
		> .q-icon
			font-size 36px
			margin-top -5px
			margin-right 10px
			&.smaller
				font-size 32px
		.q-chip
			text-transform initial
			&.main-card-chip
				background-color $light
			.q-chip-side.chip-left
				min-width 20px
				width 20px
				margin-left -4px

	.q-card-main
		font-size: 1rem

	.alert-bar
		padding 15px
		margin-bottom 16px
		border none
		&.alert-bar-info
			background-color $positive
			color $white
		&.alert-bar-warning
			background-color $negative
			color $white
		.q-icon
			font-size 24px
			margin-right 6px

	.q-tooltip.warning
		background-color $negative

	.q-collapsible > .q-item-link
		background-color $secondary
		color $white
		&:hover
			background-color $teal-4
		.q-item-side
			min-width: 20px
		i.q-item-icon
			color $white

	.main-card
		background-color $light
		border 1px solid #bcbcbc
		.q-card-primary
			background-color $tertiary
		.q-collapsible
			background-color $white

	.q-list
		padding 0

	.fieldset:not( :last-child )
		margin-bottom 24px

	.q-table
		margin-top 8px
		th
			font-weight bold
		th, td
			vertical-align top
		thead, th
			background-color $faded
			color $white
			font-size 14px
			.text-muted
				color #ccc
		.small
			font-size 14px
		&.vertical-table th
			text-align right
			font-weight normal

	.q-table + p
		margin-top 16px

	.attainments
		font-size 14px
		margin 19px 0 9px
		.q-list-header
			display block
			color $white
			background-color $faded
			padding 2px 8px
			line-height: normal
		.q-item-label
			font-size 14px
		.q-item-sublabel
			font-size 12px

	.footer
		.q-chip
			color $white
			box-shadow 0 0 0 black
			&.info
				background-color $info !important
			&.warning
				background-color $negative !important
			.q-chip-side.chip-left
				margin-right 1px
			&:hover
				.footer-chip-label
					display inline !important
			&.animate
				.footer-chip-inner
					animation pulse 1s 1 cubic-bezier(0.66, 0, 0, 1)
				.q-icon
					animation spin 1.25s 1 cubic-bezier(0.66, 0, 0, 1)

	.q-stepper-nav
		justify-content flex-end
		padding-right 6px

	.modal-content
		max-width 768px
		.q-card
			margin 0

	// animations
	@keyframes pulse
		from
			opacity 0
		to
			opacity 1

	@keyframes spin
		from
			transform rotate(0deg)
		to
			transform rotate(360deg)

	// move button onto footer on wider screen
	@media (max-device-width 500px )
		.q-stepper-nav
			right 10px
			bottom 60px
			background-color rgba(255, 255, 255, 0.9)
			padding 8px 6px
			min-height 30px
			border 1px solid rgba(200, 200, 200, 0.7)
			border-radius 3px
			box-shadow 0 0 3px rgba(0, 0, 0, 0.1), 0 1px 1px rgba(0, 0, 0, 0.12), 0 1px 1px -1px rgba(0, 0, 0, 0.12)
			.q-btn
				padding 8px !important
				font-size 12px

	@media (min-device-width 500px )
		.q-stepper-nav
			button.q-btn-flat
				.q-focus-helper
					background-color $white
				.q-btn-inner
					color $white !important

</style>
