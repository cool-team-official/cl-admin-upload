<template>
	<div class="cl-upload__wrap">
		<div
			class="cl-upload"
			:class="[
				`cl-upload--${listType}`,
				{
					'is-multiple': multiple,
					'is-drag': drag,
				},
			]"
		>
			<el-input class="cl-upload__hidden" type="hidden" v-model="value"></el-input>

			<el-upload
				:action="_action"
				:accept="_accept"
				:multiple="multiple"
				:limit="limit"
				:data="data"
				:name="_name"
				:drag="drag"
				:list-type="listType"
				:file-list="fileList"
				:show-file-list="_showFileList"
				:auto-upload="autoUpload"
				:disabled="disabled"
				:headers="{
					Authorization: token,
					...headers,
				}"
				:http-request="action ? undefined : httpRequest"
				:on-remove="_onRemove"
				:on-preview="_onPreview"
				:on-success="_onSuccess"
				:on-error="onError"
				:on-progress="onProgress"
				:on-change="onChange"
				:on-exceed="onExceed"
				:before-upload="_beforeUpload"
				:before-remove="beforeRemove"
				:style="_style"
				v-loading="_loading"
			>
				<slot>
					<!-- picture-card -->
					<template v-if="listType == 'picture-card'">
						<i :class="['cl-upload__icon', _icon]"></i>
						<span class="cl-upload__text" v-if="_text">{{ _text }}</span>
					</template>

					<!-- file -->
					<template v-else-if="listType == 'file'">
						<el-button size="mini" type="primary" :loading="loading"
							>点击上传</el-button
						>
					</template>

					<template v-else>
						<div class="cl-upload__cover" v-if="_urls[0]">
							<img v-if="_urls[0].type == 'image'" :src="_urls[0].url" />

							<span v-else>{{ _urls[0].name }}</span>
						</div>

						<template v-else>
							<i :class="['cl-upload__icon', _icon]"></i>
							<span class="cl-upload__text" v-if="_text">{{ _text }}</span>
						</template>
					</template>
				</slot>
			</el-upload>
		</div>

		<cl-dialog
			title="图片预览"
			:visible.sync="preview.visible"
			:props="{
				width: previewWidth,
			}"
		>
			<img width="100%" :src="preview.url" alt="" />
		</cl-dialog>
	</div>
</template>

<script>
import path from "path";
import { mapGetters } from "vuex";
import { baseUrl } from "@/config/env";
import { clone, last, isArray, isNumber } from "cl-admin/utils";

export default {
	name: "cl-upload",

	props: {
		value: [Array, String],
		// 上传的地址，当为 comm 时，调用 /comm/upload 接口
		action: String,
		// 尺寸
		size: [Array, String, Number],
		// 显示图标
		icon: String,
		// 显示文案
		text: String,
		// 上传的文件类型
		accept: String,
		// 上传的文件字段名
		name: String,
		// 文件列表的类型
		listType: {
			type: String,
			default: "default",
		},
		// 是否显示已上传文件列表
		showFileList: {
			type: Boolean,
			default: undefined,
		},
		// 是否自带上传
		autoUpload: {
			type: Boolean,
			default: true,
		},
		// 是否多选
		multiple: Boolean,
		// 最大允许上传个数
		limit: Number,
		// 最大允许上传文件大小(MB)
		limitSize: {
			type: Number,
			default: 2,
		},
		// 上传时附带的额外参数
		data: Object,
		// 是否拖拽
		drag: Boolean,
		// 是否禁用
		disabled: Boolean,
		// 设置上传的请求头部
		headers: Object,
		// 文件列表移除文件时的钩子
		onRemove: Function,
		// 点击文件列表中已上传的文件时的钩子
		onPreview: Function,
		// 文件上传成功时的钩子
		onSuccess: Function,
		// 文件上传失败时的钩子
		onError: Function,
		// 文件上传时的钩子
		onProgress: Function,
		// 文件状态改变时的钩子
		onChange: Function,
		// 文件超出个数限制时的钩子
		onExceed: Function,
		// 上传文件之前的钩子
		beforeUpload: Function,
		// 删除文件之前的钩子
		beforeRemove: Function,
		// 预览图片的宽度
		previewWidth: {
			type: String,
			default: "500px",
		},
	},

	data() {
		return {
			fileList: [],
			urls: [],
			loading: false,
			preview: {
				visible: false,
				url: null,
			},
		};
	},

	computed: {
		...mapGetters(["token", "components"]),

		conf() {
			return this.components.upload.options;
		},

		_action() {
			let d = this.action || this.conf.action || "oss";

			if (d == "comm") {
				return baseUrl + "/comm/upload";
			} else {
				return d;
			}
		},

		_size() {
			return this.size || this.conf.size || "128px";
		},

		_icon() {
			return this.icon || this.conf.icon || "el-icon-upload";
		},

		_text() {
			return this.text || this.conf.text || "选择文件";
		},

		_accept() {
			let d = this.accept || this.conf.accept;

			switch (this.listType) {
				case "picture-card":
					return d || "image/*";
				default:
					return d;
			}
		},

		_name() {
			return this.name || this.conf.name || "file";
		},

		_limitSize() {
			return this.limitSize || this.conf.limitSize || 2;
		},

		_showFileList() {
			let f = null;

			switch (this.listType) {
				case "picture-card":
				case "file":
					f = true;
					break;
				default:
					f = false;
					break;
			}

			return this.showFileList === undefined ? f : this.showFileList;
		},

		_loading() {
			return this.listType == "default" ? this.loading : false;
		},

		_urls() {
			const format = {
				image: ["bmp", "jpg", "jpeg", "png", "tif", "gif", "svg"],
			};

			return this.urls.map((e) => {
				let arr = e.url.split(".");
				let suf = last(arr);
				e.type = format.image.includes(suf) ? "image" : null;
				return e;
			});
		},

		_style() {
			let arr = [];

			if (isArray(this._size)) {
				arr = this._size;
			} else {
				arr = [this._size, this._size];
			}

			const [height, width] = arr.map((e) => (isNumber(e) ? `${e}px` : e));

			if (this.listType == "default" && !this.drag) {
				return {
					height,
					width,
				};
			} else {
				return {};
			}
		},
	},

	watch: {
		value: {
			immediate: true,
			handler: "parseValue",
		},
	},

	methods: {
		// 解析参数
		parseValue(value) {
			let list = [];

			if (this.multiple) {
				if (value instanceof Array) {
					list = value;
				} else {
					list = (value || "").split(",");
				}
			} else {
				if (value) {
					list = [value];
				}
			}

			// 比较数据，避免重复动画
			if (list.join(",") !== this.urls.map((e) => e.url).join(",")) {
				this.fileList = list.filter(Boolean).map((url) => {
					return {
						url,
						name: path.basename(url),
						uid: url,
					};
				});

				// 设置 URLS
				this.urls = clone(this.fileList);
			}
		},

		// 更新值
		update() {
			const urls = this.urls
				.map((e) => e.url)
				.filter(Boolean)
				.join(",");

			this.$emit("input", urls);
			this.$emit("change", urls);
		},

		// 追加文件
		append(data) {
			if (this.multiple) {
				this.urls.push(data);
			} else {
				this.urls = [data];
			}

			this.update();
		},

		// 关闭上传加载中
		done() {
			this.loading = false;
		},

		// 删除文件
		_onRemove(file) {
			this.urls = this.urls.filter((e) => e.uid != file.uid);
			this.update();

			// 删除文件之前的钩子
			if (this.onRemove) {
				this.onRemove(file, this.urls);
			}
		},

		// 预览图片
		_onPreview(file) {
			this.preview.visible = true;
			this.preview.url = file.url;

			if (!file.url) {
				let item = this.urls.find((e) => e.uid == file.uid);

				if (item) {
					this.preview.url = item.url;
				}
			}
		},

		// 上传前
		_beforeUpload(file) {
			this.loading = true;

			if (this._limitSize) {
				if (file.size / 1024 / 1024 >= this._limitSize) {
					this.$message.error(`上传文件大小不能超过 ${this._limitSize}MB!`);
					this.done();
					return false;
				}
			}

			if (this.beforeUpload) {
				return this.beforeUpload(file, {
					done: this.done,
				});
			}

			return true;
		},

		// 上传成功
		_onSuccess(res, file) {
			this.loading = false;

			this.append({
				url: res.data,
				name: file.raw.name,
				uid: file.raw.uid,
			});

			// 文件上传成功时的钩子
			if (this.onSuccess) {
				this.onSuccess(res, file.raw, this.urls);
			}
		},

		// 重设上传请求
		httpRequest(req) {
			this.loading = true;

			this.$service.common
				.ossUpload(req.file, {
					onUploadProgress: (e) => {
						if (this.onProgress) {
							e.percent = parseInt((e.loaded / e.total) * 100);
							this.onProgress(e, req.file);
						}
					},
				})
				.then((url) => {
					this._onSuccess({ data: url }, { raw: req.file });
				})
				.catch((err) => {
					console.error("upload error", err);
					this.$message.error(err);

					// 	文件上传失败时的钩子
					if (this.onError) {
						this.onError(err, req.file);
					}
				})
				.done(() => {
					this.loading = false;
				});
		},
	},
};
</script>

<style lang="scss" scoped>
.cl-upload {
	display: flex;
	flex-wrap: wrap;

	&__hidden {
		height: 0;
		width: 0;
	}

	&.is-multiple {
		.cl-upload__wrap {
			margin-right: 10px;
		}
	}

	&--default {
		&:not(.is-drag) {
			/deep/.el-upload {
				display: flex;
				align-items: center;
				justify-content: center;
				border: 1px dashed #d9d9d9;
				border-radius: 6px;
				cursor: pointer;
				position: relative;
				overflow: hidden;
				height: 100%;

				&:hover {
					border-color: #409eff;
				}

				.cl-upload__cover {
					img {
						display: block;
						height: 100%;
						width: 100%;
					}
				}

				i {
					font-size: 28px;
					color: #8c939d;
				}
			}
		}
	}

	&--picture-card {
		/deep/.el-upload {
			.cl-upload__icon {
				position: relative;
				top: 4px;
			}
		}
	}

	&__icon + span {
		margin-left: 5px;
	}

	&__text {
		font-size: 14px;
	}
}
</style>
