<template>
	<div class="x-padding flex vertical height100 note-history-dialog-vue">
		<!-- 搜索框 -->
		<div class="search-container">
			<i class="search-icon">🔍</i>
			<input type="text" class="search-input" placeholder="搜索历史记录..." />
		</div>
		<!-- 历史记录列表 -->
		<div class="history-list">
			<div class="history-item" v-for="row in configs_log_list.data.list" :key="row._id">
				<!-- <div class="history-preview" v-html="row.data"></div> -->
				<div class="history-meta flex">
					<span>{{ _.$dateFormat(row.add_time) }}</span>
					<xGap r />
					<span>{{ row.username }}</span>
					<xGap r />
					<button class="restore-btn" @click="toggleCollapse(row._id)">Diff查看</button>
					<xGap f />
				</div>
				<div class="diff-content" v-if="isShow(row._id)">
					<div class="diff-text" v-html="noteContent(row.data)" />
				</div>
			</div>
		</div>
		<xPagination :configs="configs_log_list" />
	</div>
</template>
<script lang="ts">
export default async function ({ OPTIONS: { wiki } }) {
	const typeid = wiki._id;

	return defineComponent({
		mounted() {
			this.configs_log_list.onQuery();
		},
		data(vm) {
			return {
				showContent: {},
				configs_log_list: defTable({
					async onQuery(pagination) {
						_.$loading(true);
						try {
							const { page, size } = _.$setPagination(
								vm.configs_log_list,
								pagination
							);

							const { data: list_total_data } = await _api.yapi.get_log_list({
								type: "wiki_doc",
								typeid,
								page,
								size
							});
							_.$setTableData(vm.configs_log_list, list_total_data);
						} catch (error) {
							_.$msgError(error);
						} finally {
							_.$loading(false);
						}
					},
					data: {
						list: []
					},
					columns: []
				})
			};
		},
		computed: {},
		watch: {},
		methods: {
			noteContent(content) {
				return String(content).split("\\n").join("<br/>");
			},
			toggleCollapse(id) {
				const isShow = !_.$val(this, `showContent.${id}`);
				_.$val(this, `showContent.${id}`, isShow);
			},
			isShow(id) {
				return this.showContent[id] || false;
			}
		}
	});
}
</script>
<style lang="less">
.note-history-dialog-vue {
	/* 基础样式重置 */
	* {
		margin: 0;
		padding: 0;
		box-sizing: border-box;
		font-family: "Segoe UI", Tahoma, Geneva, Verdana, sans-serif;
	}

	/* 搜索框 */
	.search-container {
		padding: 1rem;
		border-bottom: 1px solid #eee;
		position: relative;
	}

	.search-input {
		width: 100%;
		padding: 0.7rem 0.7rem 0.7rem 2.5rem;
		border: 1px solid #ddd;
		border-radius: 4px;
		font-size: 0.9rem;
	}

	.search-input:focus {
		outline: none;
		border-color: var(--el-color-primary);
	}

	.search-icon {
		position: absolute;
		left: 1.5rem;
		top: 50%;
		transform: translateY(-50%);
		color: #999;
	}

	/* 历史记录列表 */
	.history-list {
		flex: 1;
		overflow-y: auto;
		padding: 0.5rem;
	}

	/* 隐藏滚动条但保留功能 */
	.history-list::-webkit-scrollbar {
		display: none;
	}

	.history-list {
		-ms-overflow-style: none;
		scrollbar-width: none;
	}

	/* 历史记录项 */
	.history-item {
		padding: 1rem;
		border-radius: 6px;
		margin-bottom: 0.5rem;
		background: white;
		border-left: 3px solid #ddd;
		transition: all 0.2s;
	}

	.history-item:hover {
		background-color: #f9f9f9;
		transform: translateX(3px);
	}

	.history-item.current {
		border-left-color: var(--el-color-primary);
	}

	.history-meta {
		display: flex;
		justify-content: space-between;
		align-items: center;
		font-size: 0.75rem;
		color: #999;
	}

	.restore-btn {
		color: var(--el-color-primary);
		background: none;
		border: none;
		cursor: pointer;
		font-size: 0.75rem;
		transition: color 0.2s;
	}

	.restore-btn:hover {
		color: #2980b9;
	}
	/* 上下文内容 - 保持中性 */
	.jsondiffpatch-textdiff-context {
		color: #333;
		background-color: transparent;
		padding: 2px 4px;
		border-radius: 3px;
	}

	/* 删除内容 - 红色系标识 */
	.jsondiffpatch-textdiff-deleted {
		color: #721c24;
		background-color: #f8d7da;
		text-decoration: line-through;
		padding: 2px 4px;
		border-radius: 3px;
		font-weight: 500;
		border-left: 3px solid #dc3545;
		margin-left: -3px;
	}

	/* 新增内容 - 绿色系标识 */
	.jsondiffpatch-textdiff-added {
		color: #155724;
		background-color: #d4edda;
		padding: 2px 4px;
		border-radius: 3px;
		font-weight: 500;
		border-left: 3px solid #28a745;
		margin-left: -3px;
	}

	/* 增强差异行的视觉层次 */
	.jsondiffpatch-textdiff li {
		margin-bottom: 12px;
		padding: 8px;
		border-radius: 4px;
		transition: background-color 0.2s ease;

		&:has(.jsondiffpatch-textdiff-deleted) {
			background-color: rgba(248, 215, 218, 0.1);
		}

		&:has(.jsondiffpatch-textdiff-added) {
			background-color: rgba(212, 237, 218, 0.1);
		}
	}

	.jsondiffpatch-textdiff-location {
		display: inline-block;
		padding: 3px 8px;
		margin-right: 10px;
		background-color: #f1f3f5;
		border-radius: 4px;
		font-size: 12px;
		color: #495057;
		white-space: nowrap;
		border: 1px solid #e9ecef;
		box-shadow: 0 1px 2px rgba(0, 0, 0, 0.05);
	}

	/* 行号样式 */
	.jsondiffpatch-textdiff-line-number {
		font-weight: 600;
		color: #2c3e50;
		padding-right: 6px;
		border-right: 1px solid #dee2e6;
		margin-right: 6px;
		position: relative;

		&::after {
			content: "行";
			font-weight: normal;
			font-size: 11px;
			color: #868e96;
			margin-left: 3px;
		}
	}

	/* 字符位置样式 */
	.jsondiffpatch-textdiff-char {
		font-weight: 600;
		color: #795548;

		&::after {
			content: "位";
			font-weight: normal;
			font-size: 11px;
			color: #868e96;
			margin-left: 3px;
		}
	}
}
</style>
