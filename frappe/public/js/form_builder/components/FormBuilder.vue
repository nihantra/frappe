<script setup>
import Sidebar from "./Sidebar.vue";
import Tabs from "./Tabs.vue";
import { computed, onMounted, watch, ref } from "vue";
import { useStore } from "../store";
import { onClickOutside } from "@vueuse/core";

let store = useStore();

let should_render = computed(() => {
	return Object.keys(store.layout).length !== 0;
});

let container = ref(null);
onClickOutside(container, () => store.selected_field = null);

function setup_change_doctype_dialog() {
	store.page.$title_area.on("click", () => {
		let dialog = new frappe.ui.Dialog({
			title: __("Change DocType"),
			fields: [
				{
					label: __("Select DocType"),
					fieldname: "doctype",
					fieldtype: "Link",
					options: "DocType",
					default: store.doctype || null
				},
				{
					label: __("For Customize Form"),
					fieldname: "for_customize_form",
					fieldtype: "Check",
					default: store.is_customize_form
				}
			],
			primary_action_label: __("Change"),
			primary_action({ doctype }) {
				dialog.hide();
				let customize = dialog.get_value("for_customize_form") ? "customize" : "";
				frappe.set_route("form-builder", doctype, customize);
			}
		});
		dialog.show();
	});
}

watch(
	() => store.layout,
	() => (store.dirty = true),
	{ deep: true }
);

onMounted(() => {
	store.fetch();
	setup_change_doctype_dialog();
});
</script>

<template>
	<div
		v-if="should_render"
		ref="container"
		class="form-builder-container"
		@click="store.selected_field = null"
	>
		<div class="form-controls" @click.stop>
			<div class="form-sidebar">
				<Sidebar />
			</div>
		</div>
		<div class="form-container">
			<div class="form-main" :class="[store.preview ? 'preview' : '']">
				<Tabs />
			</div>
		</div>
	</div>
</template>

<style lang="scss" scoped>
.form-builder-container {
	margin-bottom: -60px;
	display: flex;
	gap: 20px;

	&.resizing {
		user-select: none;
		cursor: col-resize;
	}

	.form-controls {
		position: relative;
	}

	.form-container {
		flex: 1;
	}

	.form-sidebar,
	.form-main {
		border-radius: var(--border-radius);
		box-shadow: var(--card-shadow);
		background-color: var(--card-bg);

		:deep(.column-container .field.sortable-chosen) {
			background-color: var(--bg-light-gray);
			border-radius: var(--border-radius-sm);
			border: 1px solid transparent;
			padding: 0.3rem;
			font-size: var(--text-sm);
			cursor: pointer;

			&:has(.drop-it-here) {
				position: relative;
				background-color: transparent;
				height: 60px;

				.drop-it-here {
					display: flex;
					justify-content: center;

					&::after {
						content: "Drop it here";
						top: 31%;
						position: absolute;
						padding: 2px 10px;
						color: var(--text-dark);
						background-color: var(--gray-500);
						border-radius: var(--border-radius-full);
						z-index: 1;
					}
					&::before {
						content: "";
						top: 47%;
						position: absolute;
						width: 97%;
						height: 4px;
						background-color: var(--gray-500);
						border-radius: var(--border-radius-full);
					}
				}
			}

			&:not(:first-child) {
				margin-top: 0.4rem;
			}
		}

		:deep(.field) {
			.label {
				margin-bottom: 0.3rem;
			}

			.editable {
				input,
				textarea,
				select,
				.ace_editor,
				.ace_gutter,
				.ace_content,
				.signature-field,
				.missing-image,
				.ql-editor {
					background-color: var(--fg-color);
					cursor: pointer;
				}

				.input-text {
					background-color: inherit;
				}
			}

			.reqd::after {
				content: " *";
				color: var(--red-400);
			}
			.description,
			.time-zone {
				font-size: var(--text-sm);
				color: var(--text-muted);
			}
		}

		:deep([data-has-std-field="false"]),
		:deep([data-is-custom="1"]) {
			background-color: var(--yellow-highlight-color);
		}
	}

	:deep(.preview) {
		.tab, .column, .field, [data-is-custom="1"] {
			background-color: var(--fg-color);
		}

		.column, .field {
			border: none;
			padding: 0;
		}

		.form-section {
			padding: 5px;

			.section-header {
				&.has-label {
					padding: 10px 15px;
					margin-bottom: 8px;
				}

				&.collapsed {
					margin-bottom: 0;
				}
			}

			.section-columns {
				margin-top: 8px;

				.section-columns-container {
					.column {
						padding-left: 15px;
						padding-right: 15px;
						margin: 0;

						.field {
							margin: 0;
							margin-bottom: 1rem;
							.field-controls {
								margin-bottom: 5px;
							}
						}
					}
				}
			}
		}

		.selected, .hovered {
			border-color: transparent;
		}

		input,
		textarea,
		select,
		.ace_editor,
		.ace_gutter,
		.ace_content,
		.signature-field,
		.missing-image,
		.ql-editor {
			background-color: var(--control-bg) !important;
		}

		input[type="checkbox"] {
			background-color: var(--fg-bg) !important;
		}
	}

	.form-main:not(:has(.tab-header)) :deep(.tab-contents) {
		max-height: calc(100vh - 160px);
	}
}
</style>
