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
				<div class="history-meta flex">
					<span class="history-time">{{ _.$dateFormat(row.add_time) }}</span>
					<xGap r />
					<span class="history-author">{{ row.username }}</span>
					<xGap r />
					<button class="restore-btn" @click="toggleCollapse(row._id)">
						{{ isShow(row._id) ? "收起Diff" : "Diff查看" }}
					</button>
					<xGap f />
				</div>
				<div class="diff-content" v-if="isShow(row._id)">
					<div class="diff-text" v-html="noteContent(row.data)" />
				</div>
			</div>
			<!-- 空状态提示 -->
			<div class="empty-state" v-if="configs_log_list.data.list.length === 0">
				<div class="empty-icon">📜</div>
				<div class="empty-text">暂无历史记录</div>
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
	/* 基础样式 */
	* {
		margin: 0;
		padding: 0;
		box-sizing: border-box;
		font-family: "Segoe UI", Tahoma, Geneva, Verdana, sans-serif;
	}
	color: #333;
	background-color: #f7f8fa;

	/* 搜索框优化 */
	.search-container {
		padding: 1rem;
		background-color: #fff;
		border-bottom: 1px solid #f0f0f0;
		position: relative;
		box-shadow: 0 1px 2px rgba(0, 0, 0, 0.03);

		.search-input {
			width: 100%;
			padding: 0.7rem 0.7rem 0.7rem 2.5rem;
			border: 1px solid #e5e6eb;
			border-radius: 6px;
			font-size: 0.9rem;
			transition: all 0.2s ease;

			&:focus {
				outline: none;
				border-color: var(--el-color-primary);
				box-shadow: 0 0 0 3px rgba(64, 158, 255, 0.1);
			}
		}

		.search-icon {
			position: absolute;
			left: 1.5rem;
			top: 50%;
			transform: translateY(-50%);
			color: #909399;
			font-size: 1rem;
		}
	}

	/* 历史记录列表 */
	.history-list {
		flex: 1;
		overflow-y: auto;
		padding: 0.8rem;

		/* 滚动条美化 */
		&::-webkit-scrollbar {
			width: 6px;
			height: 6px;
		}
		&::-webkit-scrollbar-track {
			background: #f1f1f1;
			border-radius: 3px;
		}
		&::-webkit-scrollbar-thumb {
			background: #c1c1c1;
			border-radius: 3px;
		}
		&::-webkit-scrollbar-thumb:hover {
			background: #a8a8a8;
		}
	}

	/* 历史记录项 */
	.history-item {
		padding: 1rem;
		border-radius: 8px;
		margin-bottom: 0.8rem;
		background: white;
		border-left: 3px solid #e5e6eb;
		transition: all 0.25s ease;
		box-shadow: 0 1px 2px rgba(0, 0, 0, 0.04);

		&:hover {
			background-color: #fafafa;
			transform: translateX(3px);
			box-shadow: 0 2px 8px rgba(0, 0, 0, 0.06);
		}

		&.current {
			border-left-color: var(--el-color-primary);
			box-shadow: 0 2px 10px rgba(64, 158, 255, 0.1);
		}
	}

	/* 元数据样式 */
	.history-meta {
		display: flex;
		justify-content: space-between;
		align-items: center;
		font-size: 0.8rem;
		color: #868e96;
		margin-bottom: 0.8rem;

		.history-time {
			color: #6c757d;
			font-weight: 500;
		}

		.history-author {
			color: #495057;
			padding: 0 4px;
			background-color: rgba(0, 0, 0, 0.02);
			border-radius: 2px;
		}
	}

	/* 按钮样式优化 */
	.restore-btn {
		color: var(--el-color-primary);
		background: rgba(64, 158, 255, 0.05);
		border: 1px solid transparent;
		cursor: pointer;
		font-size: 0.8rem;
		transition: all 0.2s ease;
		padding: 4px 10px;
		border-radius: 4px;
		font-weight: 500;

		&:hover {
			color: #3a8ee6;
			background: rgba(64, 158, 255, 0.1);
			border-color: rgba(64, 158, 255, 0.2);
		}

		&:active {
			transform: scale(0.98);
		}
	}

	/* 差异内容样式增强 */
	.diff-content {
		margin: 12px 0 0;
		padding: 16px;
		background-color: #fcfcfc;
		border-radius: 6px;
		border: 1px solid #f0f0f0;
		overflow-x: auto;

		.jsondiffpatch-delta {
			padding: 12px;
			background-color: #fff;
			border-radius: 4px;
			border: 1px solid #f5f5f5;
		}
	}

	/* 差异文本样式 */
	.jsondiffpatch-textdiff {
		list-style: none;
		padding: 0;
		margin: 0;
	}

	.jsondiffpatch-textdiff li {
		margin-bottom: 8px;
		padding: 8px 10px;
		border-radius: 4px;
		transition: background-color 0.2s ease;

		&:last-child {
			margin-bottom: 0;
		}

		&:has(.jsondiffpatch-textdiff-deleted) {
			background-color: rgba(248, 215, 218, 0.1);
		}

		&:has(.jsondiffpatch-textdiff-added) {
			background-color: rgba(212, 237, 218, 0.1);
		}

		&:hover {
			background-color: rgba(0, 0, 0, 0.02);
		}
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

	/* 位置信息样式 */
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

	/* 空状态样式 */
	.empty-state {
		text-align: center;
		padding: 4rem 1rem;
		color: #adb5bd;

		.empty-icon {
			font-size: 3rem;
			margin-bottom: 1rem;
			opacity: 0.7;
		}

		.empty-text {
			font-size: 1rem;
			font-weight: 500;
		}
	}

	/* 响应式调整 */
	@media (max-width: 600px) {
		.search-container {
			padding: 0.8rem;
		}

		.history-list {
			padding: 0.5rem;
		}

		.history-item {
			padding: 0.8rem;
			margin-bottom: 0.6rem;
		}

		.history-meta {
			flex-wrap: wrap;
			gap: 8px 0;
			font-size: 0.75rem;

			> span {
				margin-right: 10px;
			}
		}

		.diff-content {
			padding: 10px;
			margin: 10px 0 0;
		}

		.jsondiffpatch-textdiff-location {
			display: block;
			margin-bottom: 5px;
		}
	}
}
</style>
