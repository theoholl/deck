<!--
  - SPDX-FileCopyrightText: 2019 Nextcloud GmbH and Nextcloud contributors
  - SPDX-License-Identifier: AGPL-3.0-or-later
-->
<template>
	<div>
		<ul class="labels">
			<li v-for="label in labelsSorted" :key="label.id" :class="{editing: (editingLabelId === label.id)}">
				<!-- Edit Tag -->
				<template v-if="editingLabelId === label.id">
					<form class="label-form" @submit.prevent="updateLabel(label)">
						<NcColorPicker v-model="editingLabelColor"
							class="color-picker-wrapper"
							:advanced-fields="true"
							@submit="updateColor">
							<div :style="{ backgroundColor: editingLabelColor }" class="color0 icon-colorpicker" />
						</NcColorPicker>
						<input v-model="editingLabel.title" type="text">
						<input :disabled="!editLabelObjValidated"
							type="submit"
							value=""
							class="icon-confirm">
						<NcActions>
							<NcActionButton :disabled="!editLabelObjValidated"
								icon="icon-close"
								@click="editingLabelId = null">
								{{ t('deck', 'Cancel') }}
							</NcActionButton>
						</NcActions>
					</form>
					<p v-if="!editLabelObjValidated">
						{{ missingDataLabel }}
					</p>
				</template>
				<template v-else>
					<div v-if="canManage && !isArchived" class="label-title">
						<span :style="{ backgroundColor: `#${label.color}`, color: textColor(label.color) }" @click="clickEdit(label)">{{ label.title }}</span>
					</div>
					<div v-else class="label-title">
						<span :style="{ backgroundColor: `#${label.color}`, color: textColor(label.color) }">{{ label.title }}</span>
					</div>

					<NcActions v-if="canManage && !isArchived">
						<NcActionButton icon="icon-rename" @click="clickEdit(label)">
							{{ t('deck', 'Edit') }}
						</NcActionButton>
					</NcActions>
					<NcActions v-if="canManage && !isArchived">
						<NcActionButton icon="icon-delete" @click="deleteLabel(label.id)">
							{{ t('deck', 'Delete') }}
						</NcActionButton>
					</NcActions>
				</template>
			</li>

			<li v-if="addLabel" class="editing">
				<!-- New Tag -->
				<form class="label-form" @submit.prevent="clickAddLabel">
					<NcColorPicker v-model="addLabelColor"
						class="color-picker-wrapper"
						:advanced-fields="true"
						@input="updateColor">
						<div :style="{ backgroundColor: addLabelColor }" class="color0 icon-colorpicker" />
					</NcColorPicker>
					<input v-model="addLabelObj.title" type="text">
					<input :disabled="!addLabelObjValidated"
						type="submit"
						value=""
						class="icon-confirm">
					<NcActions>
						<NcActionButton icon="icon-close" @click="addLabel=false">
							{{ t('deck', 'Cancel') }}
						</NcActionButton>
					</NcActions>
				</form>
				<p v-if="!addLabelObjValidated">
					{{ missingDataLabel }}
				</p>
			</li>
			<button v-if="canManage && !isArchived" @click="clickShowAddLabel()">
				<span class="icon-add" />{{ t('deck', 'Add a new tag') }}
			</button>
		</ul>
	</div>
</template>

<script>

import { mapGetters } from 'vuex'
import Color from '../../mixins/color.js'
import { NcColorPicker, NcActions, NcActionButton } from '@nextcloud/vue'

export default {
	name: 'TagsTabSidebar',
	components: {
		NcColorPicker,
		NcActions,
		NcActionButton,
	},
	mixins: [Color],
	data() {
		return {
			editingLabelId: null,
			editingLabel: null,
			editingLabelColor: null,
			addLabelObj: null,
			addLabel: false,
			addLabelColor: null,
			missingDataLabel: t('deck', 'Title and color are required'),
			defaultColors: ['31CC7C', '17CCC', 'FF7A66', 'F1DB50', '7C31CC', 'CC317C', '3A3B3D', 'CACBCD'],
		}
	},
	computed: {
		...mapGetters({
			labels: 'currentBoardLabels',
			canManage: 'canManage',
			isArchived: 'isArchived',
		}),
		addLabelObjValidated() {
			if (this.addLabelObj.title === '') {
				return false
			}

			if (this.colorIsValid(this.addLabelObj.color) === false) {
				return false
			}

			return true
		},
		editLabelObjValidated() {
			if (this.editingLabel.title === '') {
				return false
			}

			if (this.colorIsValid(this.editingLabel.color) === false) {
				return false
			}

			return true
		},
		labelsSorted() {
			return [...this.labels].sort((a, b) => a.title.localeCompare(b.title))
		},

	},
	methods: {
		updateColor(c) {
			if (this.editingLabel === null) {
				this.addLabelObj.color = c.substring(1, 7)
			} else {
				this.editingLabel.color = c.substring(1, 7)
			}
		},
		clickEdit(label) {
			this.editingLabelId = label.id
			this.editingLabel = Object.assign({}, label)
			this.editingLabelColor = '#' + label.color
		},
		deleteLabel(id) {
			this.$store.dispatch('removeLabelFromCurrentBoard', id)
		},
		updateLabel(label) {
			this.$store.dispatch('updateLabelFromCurrentBoard', this.editingLabel)
			this.editingLabelId = null
		},
		clickShowAddLabel() {
			this.addLabelObj = { cardId: null, color: this.defaultColors[Math.floor(Math.random() * this.defaultColors.length)], title: '' }
			this.addLabelColor = '#' + this.addLabelObj.color
			this.addLabel = true
		},
		clickAddLabel() {
			this.$store.dispatch('addLabelToCurrentBoard', this.addLabelObj)
			this.addLabel = false
			this.addLabelObj = null
			this.addLabelColor = null
		},
	},
}
</script>
<style scoped lang="scss">
	$clickable-area: var(--default-clickable-area);

	.labels li {
		display: flex;
		margin-bottom: 3px;
		padding: 3px 0;

		&:hover {
			background-color: var(--color-background-hover);
			border-radius: 3px;
		}

		.label-title {
			flex-grow: 1;
			padding: 0 10px;

			&:hover, span:hover {
				cursor: pointer;
			}

			span {
				display: inline-block;
				border-radius: 15px;
				padding: 5px 12px;
			}
		}
		&:not(.editing) button {
			width: $clickable-area;
			margin: 0 0 0 -3px;
		}

		.color-picker-wrapper {
			&, &:deep > .trigger {
				width: $clickable-area;
				padding: 3px;
				display: flex;
				align-items: stretch;
				position: relative;
				height: 36px;
			}

			.color0 {
				position: absolute;
				width: calc(#{$clickable-area} - 6px);
				height: calc(#{$clickable-area} - 6px);
				background-size: 14px;
				border-radius: 50%;
			}
		}

		&.editing {
			display: block;
		}
		form {
			display: flex;
			align-items: center;
			input[type=text] {
				flex-grow: 1;
				margin: 0 5px;
			}
		}

		p {
			padding: 0 5px;
		}
	}	
</style>
