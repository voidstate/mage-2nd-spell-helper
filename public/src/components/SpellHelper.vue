<template>
	<q-layout ref="layout">

		<!-- Header -->
		<q-toolbar slot="header" color="primary">

			<q-toolbar-title>
				Improvised Spell Helper <span slot="subtitle">Mage: The Ascension 2nd Edition</span>
			</q-toolbar-title>

			<q-btn flat>
				<q-icon name="ion-ios-information"/>
			</q-btn>
			<q-btn flat>
				<q-icon name="ion-ios-gear"/>
			</q-btn>
			<div class="toolbar-spacer gt-sm">|</div>
			<q-btn flat v-on:click="open('mailto:rpg@voidstate.com')">
				<q-icon name="ion-ios-email"/>
				<q-tooltip>
					<small>Bugs, typos, suggestions, beer? Drop me a line.</small>
				</q-tooltip>
			</q-btn>
			<q-btn flat v-on:click="open('https://github.com/voidstate')">
				<q-icon name="ion-social-github"/>
				<q-tooltip>
					<small>Visit Github repo</small>
				</q-tooltip>
			</q-btn>
		</q-toolbar>

		<!-- Stepper -->
		<q-stepper color="secondary" ref="stepper" alternative-labels contractable>

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
									<h6>Is {{ caster.arcanaName }} the Mage's Highest Arcanum?</h6>
									<q-toggle v-model="caster.isHighestArcana"/>
								</div>

								<div class="fieldset">
									<q-field helper="Spell from a Common or Inferior arcanum cost one Mana to cast (unless the spell is a Praxis).">
									<h6>Is {{ caster.arcanaName }} One of the Mage's Ruling Arcana?</h6>
									<q-toggle v-model="caster.isRulingArcana"/>
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
									<q-field helper="Praxes only require three successes for an exceptional success, and do not require a Mana if the spell if from a Common or Inferior arcanum.">
										<h6>Is the Spell a Praxis?</h6>
										<q-toggle v-model="spell.isPraxis"/>
									</q-field>
								</div>

								<div class="fieldset">
									<q-field helper="The factor that will be influenced by the successes on the spellcating dice roll.">
										<h6>Primary Factor</h6>
										<q-select v-model="spell.primaryFactor" :options="spellFactorOptions"/>
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
										<q-toggle v-model="spell.spendWillpower"/>
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
									<q-toggle v-model="subject.isResisted"/>
								</div>

								<div v-bind:class="{ hidden: !subject.isResisted }" class="fieldset">
									<q-field helper="Potency must exceed this level for the spell to take effect.">
										<h6>Withstand Rating</h6>
										<q-slider v-model="subject.withstand" :min="1" :max="10" :step="1" label-always snap/>
									</q-field>
								</div>

								<div v-bind:class="{ hidden: !subject.isResisted }" class="fieldset">
									<q-field helper="If a spell has multiple Withstand ratings (eg. a Withstood spell cast with a Sympathetic Range Attainment) it uses the highest rating, +1 for every additional rating.">
										<h6>Number of Withstand Ratings</h6>
										<q-input v-model="subject.numWithstands" type="number" :min="1"/>
									</q-field>
								</div>

							</q-card-main>
						</q-card>

					</div>

				</div>

				<q-stepper-navigation>
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
									<q-collapsible label="Standard" group="casting-time" opened>
										<p>
											<b>Ritual Casting Time</b>
										</p>
										<div v-for="castingTime in standardCastingTimeOptions">
											<q-radio v-model="spell.factors.castingTime" :val="castingTime.key" :label="castingTime.label"/>
										</div>
									</q-collapsible>

									<q-collapsible label="Advanced" group="casting-time">

										<q-field helper="Using more than one yantra (or High Speech) will increase this time.">
											<label>
												<q-radio v-model="spell.factors.castingTime" val="a1"/>
												<b>Quick casting time:</b>1 Turn </label>
										</q-field>
									</q-collapsible>

								</q-list>
							</q-card-main>
						</q-card>

					</div>

					<!-- Duration -->
					<div class="col-xs-12 col-sm-6 col-xl-4" v-bind:class="{'order-first': spell.primaryFactor === 'duration'}">

						<q-card v-bind:class="{'primary-factor': spell.primaryFactor === 'duration'}">

							<q-card-title>
								<q-icon name="ion-ios-clock" class="smaller"/>
								Duration
								<q-chip icon="ion-ios-medical" class="primary-factor-chip pull-right" v-bind:class="{ hidden: spell.primaryFactor !== 'duration' }">
									Primary Factor
								</q-chip>
							</q-card-title>

							<q-card-separator/>

							<q-card-main>
								<q-list>
									<q-collapsible label="Standard" group="duration" opened>
										<template v-if="spell.primaryFactor != 'duration'">
											<q-field helper="For additional duration, add 10 turns per additional -2 dice">
												<div v-for="duration in standardDurationOptions" :key="duration.key">
													<q-radio v-model="spell.factors.duration" :val="duration.key" :label="duration.label"/>
												</div>
											</q-field>
										</template>
										<template v-else>
											<div class="fieldset">
												<q-radio v-model="spell.factors.duration" val="s1" label="<b>Standard:</b> Each success increases the spell's duration."/>
											</div>

											<q-field helper="Additional successes add 10 turns per success.">
												<table class="q-table">
													<thead>
													<tr>
														<th>Successes</th>
														<th>Duration</th>
													</tr>
													</thead>
													<tbody>
													<tr v-for="duration in standardDurationOptions" :key="duration.key">
														<td>{{ duration.successes }}</td>
														<td>{{ duration.time }}</td>
													</tr>
													</tbody>
												</table>
											</q-field>

										</template>
									</q-collapsible>

									<q-collapsible label="Advanced" group="duration">

										<template v-if="spell.primaryFactor != 'duration'">
											<div v-for="duration in advancedDurationOptions" :key="duration.key">
												<q-radio v-model="spell.factors.duration" :val="duration.key" :label="duration.label"/>
											</div>
										</template>
										<template v-else>
											<div class="fieldset">
												<q-radio v-model="spell.factors.duration" val="a1" label="<b>Advanced:</b> Each success increases the spell's duration."/>
											</div>

											<table class="q-table">
												<thead>
												<tr>
													<th>Successes</th>
													<th>Duration</th>
												</tr>
												</thead>
												<tbody>
												<tr v-for="duration in advancedDurationOptions" :key="duration.key">
													<td>{{ duration.successes }}</td>
													<td>{{ duration.time }}</td>
												</tr>
												</tbody>
											</table>

										</template>

									</q-collapsible>

								</q-list>
							</q-card-main>
						</q-card>

					</div>

					<!-- Potency -->
					<div class="col-xs-12 col-sm-6 col-xl-4" v-bind:class="{'order-first': spell.primaryFactor === 'potency'}">

						<q-card v-bind:class="{'primary-factor': spell.primaryFactor === 'potency'}">
							<q-card-title>
								<q-icon name="ion-ios-flame"/>
								Potency
								<q-chip icon="ion-ios-medical" class="primary-factor-chip pull-right" v-bind:class="{ hidden: spell.primaryFactor !== 'potency' }">
									Primary Factor
								</q-chip>
							</q-card-title>

							<q-card-separator/>

							<q-card-main>

								<div class="alert-bar alert-bar-warning" v-bind:class="{ hidden: totalWithstand === 0 }">
									<q-icon name="ion-android-warning"/>
									Potency must exceed the subject's Withstand of {{ totalWithstand }}
								</div>

								<q-list>
									<q-collapsible label="Standard" group="potency" opened>
										<div>
											<template v-if="spell.primaryFactor != 'potency'">
												<div v-for="potency in standardPotencyOptions" :key="potency.key">
													<q-radio v-model="spell.factors.potency" :val="potency.key" :label="potency.label" :disable="potency.value <= totalWithstand"/>
												</div>
											</template>
											<template v-else>
												<q-radio v-model="spell.factors.potency" val="s1" label="<b>Standard:</b> Each success provides 1 Potency."/>
											</template>
										</div>
									</q-collapsible>
									<q-collapsible label="Advanced" group="potency">
										<div>
											<template v-if="spell.primaryFactor != 'potency'">
												<q-field helper="Advanced Potency grants an additional -2 to Withstand.">
													<div v-if="spell.primaryFactor != 'potency'" v-for="potency in advancedPotencyOptions" :key="potency.key">
														<q-radio v-model="spell.factors.potency" :val="potency.key" :label="potency.label" :disable="potency.value <= totalWithstand"/>
													</div>
												</q-field>
											</template>
											<template v-else>
												<q-field helper="Advanced Potency grants an additional -2 to Withstand.">
													<q-radio v-model="spell.factors.potency" val="a1" label="<b>Advanced:</b> Each success provides 1 Potency."/>
												</q-field>
											</template>

										</div>
									</q-collapsible>

								</q-list>
							</q-card-main>
						</q-card>

					</div>

					<!-- Range -->
					<div class="col-xs-12 col-sm-6 col-xl-4">

						<q-card v-bind:class="{'primary-factor': spell.primaryFactor === 'range'}">
							<q-card-title>
								<q-icon name="ion-ios-eye"/>
								Range
							</q-card-title>

							<q-card-separator/>

							<q-card-main>
								<q-list>
									<q-collapsible label="Standard" group="range" opened>
										<label>
											<q-radio v-model="spell.factors.range" val="s1"/>
											<b>Standard:</b> Self/touch or Aimed </label>
									</q-collapsible>
									<q-collapsible label="Advanced" group="range">
										<label>
											<q-radio v-model="spell.factors.range" val="a1"/>
											<b>Advanced:</b> Sensory </label>
									</q-collapsible>

								</q-list>
							</q-card-main>
						</q-card>

					</div>

					<!-- Scale -->
					<div class="col-xs-12 col-sm-6 col-xl-4" v-bind:class="{'order-first': spell.primaryFactor === 'scale'}">

						<q-card v-bind:class="{'primary-factor': spell.primaryFactor === 'scale'}">

							<q-card-title icon="ion-ios-people-outline">
								<q-icon name="ion-ios-people"/>
								Scale
								<q-chip icon="ion-ios-medical" class="primary-factor-chip pull-right" v-bind:class="{ hidden: spell.primaryFactor !== 'scale' }">
									Primary Factor
								</q-chip>
							</q-card-title>

							<q-card-separator/>

							<q-card-main>
								<q-list>
									<q-collapsible label="Standard" group="scale" opened>
										<template v-if="spell.primaryFactor != 'scale'">

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

										</template>
										<template v-else>
											<div class="fieldset">
												<q-radio v-model="spell.factors.scale" val="s1" label="<b>Standard:</b> Each success increases the spell's scale."/>
											</div>

											<table class="q-table q-table highlight responsive compact">
												<thead>
												<tr>
													<th class="text-left">Successes</th>
													<th class="text-left">Number of Subjects</th>
													<th class="text-left">Size of Largest Subject</th>
													<th class="text-left">Area of Effect</th>
												</tr>
												</thead>
												<tbody>
												<tr v-for="scale in standardScaleOptions" :key="scale.key">
													<td>{{ scale.value }}</td>
													<td data-th="Number of Subjects">{{ scale.number }}</td>
													<td data-th="Size of Largest Subject">{{ scale.size }}</td>
													<td data-th="Area of Effect">
														<small>{{ scale.area }}</small>
													</td>
												</tr>
												</tbody>
											</table>

										</template>
									</q-collapsible>

									<q-collapsible label="Advanced" group="scale">
										<template v-if="spell.primaryFactor != 'scale'">
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
										</template>
										<template v-else>
											<div class="fieldset">
												<q-radio v-model="spell.factors.scale" val="a1" label="<b>Advanced:</b> Each success increases the spell's scale."/>
											</div>

											<table class="q-table highlight responsive compact">
												<thead>
												<tr>
													<th class="text-left">Successes</th>
													<th class="text-left">Number of Subjects</th>
													<th class="text-left">Size of Largest Subject</th>
													<th class="text-left">Area of Effect</th>
												</tr>
												</thead>
												<tbody>
												<tr v-for="scale in advancedScaleOptions" :key="scale.key">
													<td>{{ scale.value }}</td>
													<td data-th="Number of Subjects">{{ scale.number }}</td>
													<td data-th="Size of Largest Subject">{{ scale.size }}</td>
													<td data-th="Area of Effect" class="small">{{ scale.area }}</td>
												</tr>
												</tbody>
											</table>

										</template>
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
					<small>Use locations, actions and tools to help form the spell's Imago.</small>
				</h5>

				<div class="alert-bar alert-bar-info">
					<q-icon name="ion-ios-information"/>
					Gnosis {{ caster.gnosis }} allows the use of {{ maxYantras }} yantras
				</div>

				<div class="row sm-gutter">

					<div class="col-xs-12 col-sm-6 col-xl-4">

						<q-card>
							<q-card-title>
								Locations
							</q-card-title>

							<q-card-separator/>

							<q-card-main>
								<div v-for="yantra in locationYantraOptions" :key="yantra.key" class="fieldset">
									<q-field :helper="yantra.desc">
										<q-checkbox v-model="spell.yantras" :val="yantra.key" :label="yantra.label"/>
									</q-field>
								</div>
							</q-card-main>
						</q-card>

					</div>

					<div class="col-xs-12 col-sm-6 col-xl-4">

						<q-card>
							<q-card-title>
								Actions
							</q-card-title>

							<q-card-separator/>

							<q-card-main>
								<div v-for="yantra in actionYantraOptions" :key="yantra.key" class="fieldset">
									<q-field :helper="yantra.desc">
										<template v-if="yantra.key === 'a1'">
											<q-checkbox v-model="spell.yantras" :val="yantra.key" :label="yantra.label" :disable="!isConcentrationMantraAllowed">

												<q-tooltip class="warning" v-bind:class="{ hidden: isConcentrationMantraAllowed }">
													Duration must be more than 1 turn
												</q-tooltip>
											</q-checkbox>
										</template>
										<template v-else>
											<q-checkbox v-model="spell.yantras" :val="yantra.key" :label="yantra.label"/>
										</template>
									</q-field>
								</div>
							</q-card-main>
						</q-card>

					</div>

					<div class="col-xs-12 col-sm-6 col-xl-4">

						<q-card>
							<q-card-title>
								Tools
							</q-card-title>

							<q-card-separator/>

							<q-card-main>
								<div v-for="yantra in toolYantraOptions" :key="yantra.key" class="fieldset">
									<q-field :helper="yantra.desc">
										<q-checkbox v-model="spell.yantras" :val="yantra.key" :label="yantra.label"/>
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
										<q-toggle v-model="paradox.inured"/>
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
									<q-toggle v-model="paradox.sleepers"/>
								</div>

								<div class="fieldset" v-show="paradox.sleepers">
									<q-field helper="Multiple Sleeper witnesses do not add Paradox dice, but increase the chances of a Paradox occurring. If a few Sleepers witness the magic casting, the Paradox roll gains the 9-Again quality, a large group grants the Paradox roll the 8-Again quality, and a full crowd grants the Paradox roll the rote quality.">
										<h6>How Many Sleeper Witnesses?</h6>
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
										<q-toggle v-model="isDedicatedToolYantraUsed"/>
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
									<q-field helper="Mitigates 1 Paradox Dice per Mana spent. Limited by Gnosis.">
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

								<div class="alert-bar alert-bar-info" v-bind:class="{ hidden: hasParadox }">
									<q-icon name="ion-ios-checkmark"/>
									This spell does not require a Paradox roll.
								</div>

								<div v-bind:class="{ hidden: !hasParadox }">
									<div class="alert-bar alert-bar-warning">
										<q-icon name="ion-alert-circled"/>
										This spell requires a Paradox roll. Roll for Paradox first.
									</div>

									<h6>Dice pool</h6>
									<p>
										{{ paradoxDiceSummary }}
										<q-btn small flat @click="showParadoxManaModal()" title="Add">
											<q-icon name="ion-ios-plus"></q-icon>
										</q-btn>
									</p>

									<p class="text-muted">
										<small>For Paradox roll results, see Mage: the Awakening 2nd Edition, p. 115</small>
									</p>
								</div>

							</q-card-main>
						</q-card>

						<q-card v-show="spell.yantras.length > 0">
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

						<q-card>
							<q-card-title>
								Spell
							</q-card-title>

							<q-card-separator/>

							<q-card-main>

								<div class="alert-bar alert-bar-warning" v-bind:class="{ hidden: isCastable }">
									<q-icon name="ion-alert-circled"/>
									This spell is too complex and automatically fails.
								</div>

								<div v-bind:class="{ hidden: !isCastable }">

									<div class="alert-bar alert-bar-warning" v-show="isPrimaryFactor( 'potency' ) && subject.isResisted">
										<q-icon name="ion-alert-circled"/>
										This spell is Withstood. Potency {{ totalWithstand + 1 }} is required to affect the target.
									</div>

									<div class="alert-bar alert-bar-info" v-bind:class="{ hidden: !spell.isPraxis }">
										<q-icon name="ion-ios-information"/>
										This spell only requires three successes for an exceptional success.
									</div>

									<h6>Arcanum</h6>
									<p>
										{{ caster.arcanaName }}
										<span v-for="i in spell.arcana" class="arcana-dot">&#9679;</span>
									</p>

									<h6>Mana Cost</h6>
									<p>{{ totalMana }} </p>

									<h6>Dice Pool</h6>
									<p>{{ dicePoolSummary }}
										<q-btn small flat @click="showExtraDiceModal()" title="Add">
											<q-icon name="ion-ios-plus"></q-icon>
										</q-btn>
									</p>

									<h6>Primary Factor</h6>
									<p>{{ primaryFactorLabel }}</p>

									<h6>Successes</h6>

									<p>A single success means the spell takes place.</p>

									<!-- Primary Duration -->
									<q-field v-show="isPrimaryFactor( 'duration' ) && !isAdvanced( 'duration')" helper="Additional successes add 10 turns per success.">
										<table class="q-table">
											<thead>
											<tr>
												<th>Successes</th>
												<th>Duration</th>
											</tr>
											</thead>
											<tbody>
											<tr v-for="duration in standardDurationOptions" :key="duration.key">
												<td>{{ duration.successes }}</td>
												<td>{{ duration.time }}</td>
											</tr>
											</tbody>
										</table>
									</q-field>
									<table class="q-table" v-show="isPrimaryFactor( 'duration' ) && isAdvanced( 'duration')">
										<thead>
										<tr>
											<th>Successes</th>
											<th>Duration</th>
										</tr>
										</thead>
										<tbody>
										<tr v-for="duration in advancedDurationOptions" :key="duration.key">
											<td>{{ duration.successes }}</td>
											<td>{{ duration.time }}</td>
										</tr>
										</tbody>
									</table>

									<!-- Primary Potency -->
									<p v-show="isPrimaryFactor( 'potency' )">
										Each success provides 1 Potency.
										<template v-show="isPrimaryFactor( 'potency' ) && isAdvanced( 'potency')">
											Advanced Potency grants an additional -2 to Withstand.
										</template>
									</p>

									<!-- Primary Scale -->
									<table v-show="isPrimaryFactor( 'scale' ) && !isAdvanced( 'scale')" class="q-table q-table highlight responsive compact">
										<thead>
										<tr>
											<th class="text-left">Successes</th>
											<th class="text-left">Number of Subjects</th>
											<th class="text-left">Size of Largest Subject</th>
											<th class="text-left">Area of Effect</th>
										</tr>
										</thead>
										<tbody>
										<tr v-for="scale in standardScaleOptions" :key="scale.key">
											<td data-th="Successes">{{ scale.value }}</td>
											<td data-th="Number of Subjects">{{ scale.number }}</td>
											<td data-th="Size of Largest Subject">{{ scale.size }}</td>
											<td data-th="Area of Effect" class="small">{{ scale.area }}</td>
										</tr>
										</tbody>
									</table>
									<q-field v-show="isPrimaryFactor( 'scale' ) && isAdvanced( 'scale')" helper="Additional scale doubles the number of subjects and adds 5 size per additional -2 dice penalty.">
										<table class="q-table highlight responsive compact">
											<thead>
											<tr>
												<th class="text-left">Successes</th>
												<th class="text-left">Number of Subjects</th>
												<th class="text-left">Size of Largest Subject</th>
												<th class="text-left">Area of Effect</th>
											</tr>
											</thead>
											<tbody>
											<tr v-for="scale in standardScaleOptions" :key="scale.key">
												<td data-th="Successes">{{ scale.value }}</td>
												<td data-th="Number of Subjects">{{ scale.number }}</td>
												<td data-th="Size of Largest Subject">{{ scale.size }}</td>
												<td data-th="Area of Effect" class="small">{{ scale.area }}</td>
											</tr>
											</tbody>
										</table>
									</q-field>

									<p class="text-muted">
										<small>For exceptional successes, see Mage: the Awakening 2nd Edition, p. 115</small>
									</p>

									<!-- Others -->
									<h6>Other Factors</h6>
									<table class="q-table vertical-table">
										<tbody>
										<tr>
											<th>Casting Time</th>
											<td>
												{{ castingTimeSummary }}
											</td>
										</tr>
										<tr>
											<th v-show="!isPrimaryFactor( 'duration' )">Duration</th>
											<td v-show="!isPrimaryFactor( 'duration' )">
												{{ durationSummary }}
											</td>
										</tr>
										<tr>
											<th v-show="!isPrimaryFactor( 'potency' )">Potency</th>
											<td v-show="!isPrimaryFactor( 'potency' )">
												{{ potencySummary }}
											</td>
										</tr>
										<tr>
											<th>Range</th>
											<td>
												{{ rangeSummary }}
											</td>
										</tr>
										<tr>
											<th v-show="!isPrimaryFactor( 'scale' )">Scale</th>
											<td v-show="!isPrimaryFactor( 'scale' )">
												{{ scaleSummary }}
											</td>
										</tr>
										</tbody>
									</table>

									<p class="text-muted">
										<small>As a general rule, each point of Potency grants a one-die bonus or penalty, deals one point of weapon damage, or heals one wound.</small>
									</p>

								</div>

							</q-card-main>
						</q-card>

					</div>

					<q-stepper-navigation>
						<q-btn color="secondary" @click="stepFirst()">Restart</q-btn>
						<q-btn color="secondary" flat @click="stepPrevious()">Back</q-btn>
					</q-stepper-navigation>

				</div>
			</q-step>
		</q-stepper>

		<!-- Footer -->
		<q-toolbar slot="footer" class="fixed-bottom">
			<q-toolbar-title>
				<q-chip icon="ion-ios-bolt-outline" class="info" small>
					<span class="footer-chip-label gt-xs">Reach</span> {{ usedReach }}/{{ freeReach }}
				</q-chip>
				<q-chip icon="ion-ios-analytics-outline" class="info" v-bind:class="{ warning: isDicePoolTooLow }" small>
					<span class="footer-chip-label gt-xs">Dice pool</span> {{ dicePool }}
					<q-tooltip v-if="isDicePoolTooLow">
						If the dice pool, after Yantras, is -6 or less, the spell is impossible
					</q-tooltip>
				</q-chip>
				<q-chip icon="ion-ios-pulse" class="info" small>
					<span class="footer-chip-label gt-xs">Mana</span> {{ totalMana }}
				</q-chip>
				<q-chip icon="ion-ios-flower" class="warning" small v-bind:class="{ hidden: !hasParadox }">
					<span class="footer-chip-label gt-xs">Paradox</span> {{ paradoxDice }}
					<q-tooltip>
						You have exceeded the free reach from your Arcanum and may cause Paradox
					</q-tooltip>
				</q-chip>
			</q-toolbar-title>
		</q-toolbar>
	</q-layout>
</template>

<script>
	import {
		// Quasar components
		QLayout,
		QToolbar, QToolbarTitle,
		QChip,
		QStepper, QStep, QStepperNavigation,
		QCard, QCardTitle, QCardSeparator, QCardMain,
		QList,
		QItem, QItemSide, QItemMain,
		QCollapsible,
		QBtn,
		QTooltip,
		QIcon,
		QField, QSlider, QSelect, QToggle, QRadio, QInput, QCheckbox,
		// Quasar utilities
		Dialog,
		Toast,
		openURL,
		debounce
	} from 'quasar'

	import _ from 'lodash'

	import {AddressbarColor} from 'quasar'

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
		'Time',
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

	const yantras = new Map( [

		// locations.
		[ 'l1', {
			name: 'Demesne',
			desc: 'A prepared ritual space with a soul stone',
			bonus: 2
		} ],
		[ 'l2', {
			name: 'Location',
			desc: 'A place and time symbolically linked to the spell.',
			bonus: 2
		} ],
		[ 'l3', {
			name: 'Supernal Verge',
			desc: 'A place where the Supernal touches the Fallen World.',
			bonus: 2
		} ],

		// actions
		[ 'a1', {
			name: 'Concentration',
			desc: 'Duration must be greater than a turn. If the mage is hurt or takes a non-reflexive action while the spell is active, it ends immediately.',
			bonus: 2
		} ],
		[ 'a2', {
			name: 'Mantra (High Speech)',
			desc: 'Must be spoken aloud. Cannot be used reflexively.',
			bonus: 2
		} ],
		[ 'a3', {
			name: 'Runes',
			desc: 'The subject is marked with runes. Ritual casting only. If anything damages or disrupts the runes while the spell is active, it ends immediately.',
			bonus: 2
		} ],

		// tools
		[ 't9', {
			name: 'Dedicated Tool',
			desc: 'An item that synchronizes with her Nimbus and that feeds in to her understanding of magic. Also reduces Paradox by 2 dice.',
			bonus: 2
		} ],
		[ 't1', {
			name: 'Order Tool',
			desc: 'Tools which draw upon an Order’s symbols rather than those of the Supernal world directly, focusing magic in a way that matches their teachings.',
			bonus: 1
		} ],
		[ 't2', {
			name: 'Material Sympathy',
			desc: 'An item sympathetically linked to the subject <i>as they are now</i>. At least one sympathetic tool is required for sympathetic casting.',
			bonus: 2
		} ],
		[ 't3', {
			name: 'Representational Sympathy',
			desc: 'An item sympathetically linked to the subject <i>as they were previously</i>. At least one sympathetic tool is required for sympathetic casting.',
			bonus: 1
		} ],
		[ 't4', {
			name: 'Symbolic Sympathy',
			desc: 'An indirect representation of the subject. At least one sympathetic tool is required for sympathetic casting.',
			bonus: 0
		} ],
		[ 't5', {
			name: 'Sacrament',
			desc: 'An object symbolic of the spell that the mage destroys during casting.',
			bonus: 1
		} ],
		[ 't6', {
			name: 'Rare Sacrament',
			desc: 'A sacrament which requires significant effort to acquire.',
			bonus: 2
		} ],
		[ 't7', {
			name: 'Otherworldly Sacrament',
			desc: 'A sacrament from somewhere other than the material realm.',
			bonus: 3
		} ],
		[ 't8', {
			name: 'Persona',
			desc: 'A persona Yantra keys in to the mage’s Shadow Name and Cabal Theme Merits.',
			bonus: 1
		} ],
	] )

	// Toast Defaults
	Toast.setDefaults( {
		icon: 'ion-ios-lightbulb',
		timeout: 3500
	} )

	export default {
		name: 'spell-helper',
		components: {
			QLayout,
			QToolbar, QToolbarTitle,
			QChip,
			QStepper, QStep, QStepperNavigation,
			QCard, QCardTitle, QCardSeparator, QCardMain,
			QList,
			QItem, QItemSide, QItemMain,
			QCollapsible,
			QBtn,
			QTooltip,
			QIcon,
			QField, QSlider, QSelect, QToggle, QRadio, QInput, QCheckbox,
		},
		data: function ()
		{
			return {
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
					bonusDice: 0,
					spendWillpower: false,
					extraReach: 0,
					yantras: []
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
					sleeperGroupSize: 'sm',
					manaSpent: 0
				}
			}
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
			freeReach()
			{
				return this.caster.arcana - this.spell.arcana + 1
			},
			usedReach()
			{
				let reach = 0

				if( this.caster.activeSpells >= this.caster.gnosis )
				{
					reach += this.caster.activeSpells - this.caster.gnosis + 1
				}

				// check factors (advanced factor keys begin with "a")
				for ( let factor of factors )
				{
					if ( this.spell.factors[ factor ][ 0 ] === 'a' )
					{
						reach += 1
					}
				}

				// indefinite duration costs 1 reach
				if ( !this.isPrimaryFactor( 'duration' ) && this.spell.factors.duration === 'a6' )
				{
					reach += 1
				}

				// spell-specific extra reach
				reach += this.spell.extraReach

				return reach
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
				if( !this.isAdvanced( 'castingTime' ) )
				{
					pool += this.spell.factors.castingTime[1] - 1
				}

				// potency
				pool -= ( this.getPotencyFromKey( this.spell.factors.potency ) - 1 ) * 2

				// duration
				pool -= durations.get( this.spell.factors.duration ).penalty

				// scale
				pool -= scales.get( this.spell.factors.scale ).penalty

				// yantras
				for ( let key of this.spell.yantras )
				{
					pool += yantras.get( key ).bonus
				}

				return pool
			},
			isDicePoolTooLow()
			{
				return this.dicePool < -5
			},
			maxCasterArcana()
			{
				let arcana

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

				if ( !this.caster.isHighestArcana )
				{
					arcana -= 1
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
			spellFactorOptions()
			{
				let options = []

				for ( let factor of factors )
				{
					if ( factor !== 'castingTime' && factor !== 'range' )
					{
						options.push( {
							label: _.upperFirst( factor ),
							value: factor
						} )
					}
				}

				return options
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
						return this.spell.yantras.includes( 't9' )
					},
					// setter
					set( isUsed )
					{
						if ( isUsed )
						{
							this.spell.yantras.push( 't9' )
						}
						else
						{
							_.pull( this.spell.yantras, 't9' )

							this.$forceUpdate() // cache: false and this.$forceUpdate() means the getter shows updated value
						}
					}
				},
			totalMana()
			{
				let mana = 0;

				if ( !this.caster.isRulingArcana && !this.spell.isPraxis )
				{
					mana++
				}

				if ( !this.isPrimaryFactor( 'duration' ) && this.spell.factors.duration === 'a6' )
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
			isCastable()
			{
				return this.dicePool > -5
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

				while ( i++ < 10 )
				{
					options.push( {
						key: 's' + i,
						value: i,
						label: `${i} (-${(i - 1) * 2} dice)`
					} )
				}

				return options
			},
			advancedPotencyOptions()
			{
				let options = []
				let i = 0
				while ( i++ < 10 )
				{
					options.push( {
						key: 'a' + i,
						value: i,
						label: `${i} (-${(i - 1) * 2} dice)`
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
						options.push( {
							key: key,
							successes: key[ 1 ],
							time: duration.time,
							label: `${duration.time} (-${duration.penalty} dice)`
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
						options.push( {
							key: key,
							successes: key[ 1 ],
							time: duration.time,
							label: `${duration.time} (-${duration.penalty} dice)`
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
				let options = []

				for ( let [ key, yantra ] of yantras )
				{
					if ( key[ 0 ] === 'l' )
					{
						let y = _.clone( yantra )
						y.key = key
						y.label = `${yantra.name} (+${yantra.bonus} ${yantra.bonus === 1 ? 'die' : 'dice'})`
						options.push( y )
					}
				}

				return options
			},
			actionYantraOptions()
			{
				let options = []

				for ( let [ key, yantra ] of yantras )
				{
					if ( key[ 0 ] === 'a' )
					{
						let y = _.clone( yantra )
						y.key = key
						y.label = `${yantra.name} (+${yantra.bonus} ${yantra.bonus === 1 ? 'die' : 'dice'})`
						options.push( y )
					}
				}

				return options
			},
			toolYantraOptions()
			{
				let options = []

				for ( let [ key, yantra ] of yantras )
				{
					if ( key[ 0 ] === 't' )
					{
						let y = _.clone( yantra )
						y.key = key
						y.label = `${yantra.name} (+${yantra.bonus} ${yantra.bonus === 1 ? 'die' : 'dice'})`
						options.push( y )
					}
				}

				return options
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
					else
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
				if ( this.spell.factors.castingTime === 's1' )
				{
					return this.baseCastingTime
				}
				else
				{
					let turns = this.numYantras <= 1 ? 1 : this.numYantras - 1;

					if ( this.spell.yantras.includes( 'a2' ) )
					{
						turns++
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
				return this.getPotencyFromKey( this.spell.factors.potency )
			},
			rangeSummary()
			{
				if ( this.spell.factors.range === 's1' )
				{
					return 'Touch/aimed'
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

				for ( let key of this.spell.yantras )
				{
					yantrasNames.push( yantras.get( key ) )
				}

				return yantrasNames
			}
		},
		watch: {
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
							Toast.create( 'Spell\'s Arcana cannot exceed Caster\'s Arcana of ' + this.caster.arcana )
						},
						500
					)()
				}
			},
			isConcentrationMantraAllowed()
			{
				if ( !this.isConcentrationMantraAllowed && this.spell.yantras.includes( 'a1' ) )
				{
					_.pull( this.spell.yantras, 'a1' )

					//this.$forceUpdate() // cache: false and this.$forceUpdate() means the getter shows updated value
				}
			},
			// disable potency checkboxes below Withstand level
			totalWithstand()
			{
				let potency = this.spell.factors.potency[ 1 ],
				    prefix  = this.spell.factors.potency[ 0 ]

				if ( potency <= this.totalWithstand )
				{
					this.spell.factors.potency = prefix + ( this.totalWithstand + 1 )
				}
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
			}
		},
		methods: {
			open( url )
			{
				openURL( url )
			},
			getPotencyFromKey( key )
			{
				return key[ 1 ]
			},
			isAdvanced( factorName )
			{
				return this.spell.factors[ factorName ][ 0 ] === 'a'
			},
			isPrimaryFactor( factorName )
			{
				return this.spell.primaryFactor === factorName
			},
			stepNext()
			{
				this.$refs.stepper.next()
				this.scrollUp()
			},
			stepPrevious()
			{
				this.$refs.stepper.previous()
				this.scrollUp()
			},
			stepFirst()
			{
				this.$refs.stepper.goToStep( 'first' )
				this.scrollUp()
			},
			scrollUp()
			{
				window.scrollTo( 0, 0 );
			},
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
								value: true
							} ],
							model: [
								this.spell.spendWillpower
							],
							withLabel: true
						}
					},
					buttons: [
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
			},
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
		}
	}
</script>

<style lang="stylus">
	@import '~variables'

	.text-muted
		color $faded

	.layout-page
		margin-bottom 40px

	.toolbar-spacer
		padding 0 1rem

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

	.sm-gutter > div > .q-card
		margin 0
		& + .q-card
			margin-top 16px

	h5
		small
			display inline-block
			font-size 1rem
			font-weight 400
			line-height 110%
			color #777

	h6
		font-size 18px

	p > small {
		font-size 14px
		line-height 11px
	}

	p .q-btn.inline
		margin-top -3px
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
			&.primary-factor-chip
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

	.q-item-link
		background-color $secondary
		color $white
		&:hover
			background-color $teal-4
		.q-item-side
			min-width: 20px
		i.q-item-icon
			color $white

	.q-card-primary
		background-color $primary
		color $white

	.primary-factor
		background-color $light
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
			font-weight normal
		th, td
			vertical-align top
		thead, th
			background-color $faded
			color $white
		.small
			font-size 85%
		&.vertical-table th
			text-align right

	.q-table + p
		margin-top 16px

	footer
		.q-chip
			color $white
			&.info
				background-color $info !important
			&.warning
				background-color $negative !important
			.q-chip-side.chip-left
				margin-right 1px
			&:hover
				.footer-chip-label
					display: inline !important

	.q-stepper-nav
		justify-content flex-end
		padding-right 6px

</style>