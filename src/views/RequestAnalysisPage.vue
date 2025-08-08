<template>
    <div class="request-analysis-page">
        <n-space vertical size="large" style="width: 100%">
            <!-- ===== 响应展示区域 ===== -->
            <n-card title="📥 插件信息" size="small">
                <n-space vertical size="medium">
                    <!-- FilePath -->
                    <n-input v-model:value="pluginRequest.filepath" placeholder="请输入插件地址 FilePath (支持 http / https)"
                        size="medium" style="width: 100%">
                        <template #prefix>
                            <n-icon>
                                <LinkIcon />
                            </n-icon>
                        </template>
                    </n-input>

                    <!-- 发送请求按钮 -->
                    <n-button type="primary" size="medium" @click="showPlugin(pluginRequest.filepath)"
                        :loading="loading" block>
                        🚀 发送请求
                    </n-button>
                </n-space>

                <div>
                    <!-- 响应状态 -->
                    <n-alert type="success" title="其他信息 (Plugin Other Information)" :closable="false">
                        <div>是否被禁用 <n-switch v-model:value="plugin.exclude" disabled /></div>
                    </n-alert>

                    <!-- 响应头 -->
                    <div style="margin-top: 16px">
                        <h4>📋 其他字段 (Plugin Other Attribute)</h4>
                        <n-dynamic-tags :value="formatHeadersForDisplay(plugin)" :read-only="true"
                            tag-props="{ size: 'small' }" />
                    </div>

                    <!-- 响应体 -->
                    <div style="margin-top: 16px">
                        <h4>📦 插件内容 (Plugin Body)</h4>
                        <n-input :value="plugin.body" type="textarea" size="medium"
                            style="width: 100%; min-height: 200px" readonly />
                    </div>
                </div>
            </n-card>

            <!-- ===== 其他功能 ===== -->
            <n-card title="📋 其他功能" size="small">
                <div style="margin-top: 16px">
                    <h4>上传插件与远程下载插件 (Upload plugin and download plugin)</h4>
                    <!-- 简单的文件上传 -->
                    <n-input v-model:value="pluginRequest.downloadPluginUrl"
                        placeholder="远程下载插件地址 PluginUrl (支持 http / https)" size="medium" style="width: 100%">
                        <template #prefix>
                            <n-icon>
                                <LinkIcon />
                            </n-icon>
                        </template>
                    </n-input>

                    <!-- 简单的文件上传 -->
                    <n-input v-model:value="pluginRequest.nameEncoding" placeholder="压缩包编码，例如 utf8 | gbk" size="medium"
                        style="width: 30%">
                        <template #prefix>
                            <n-icon>
                                <LinkIcon />
                            </n-icon>
                        </template>
                    </n-input>
                    <n-input v-model:value="pluginRequest.zipFilePath"
                        placeholder="压缩包路径，当为空会选选择上传的路径，为'download'会选择下载的路径" size="medium" style="width:70%">
                        <template #prefix>
                            <n-icon>
                                <LinkIcon />
                            </n-icon>
                        </template>
                    </n-input>

                    <n-upload style="margin-top: 10px;" v-model:file-list="fileList" :max="1" :accept="'.zip'"
                        @change="handleFileChange">
                        <n-button>选择 ZIP 文件</n-button>
                    </n-upload>

                    <n-input v-model:value="pluginRequest.fromPath" placeholder="对比路径源，默认为解压目录" size="medium"
                        style="width: 100%">
                        <template #prefix>
                            <n-icon>
                                <LinkIcon />
                            </n-icon>
                        </template>
                    </n-input>
                    <n-input v-model:value="pluginRequest.toPath" placeholder="对比路径目标，默认为插件目录" size="medium"
                        style="width: 100%">
                        <template #prefix>
                            <n-icon>
                                <LinkIcon />
                            </n-icon>
                        </template>
                    </n-input>
                </div>

                <div style="margin-top: 16px">
                    <h4>功能选项 (Plugin options)</h4>
                    <div>
                        <n-alert type="" title="" :closable="false">
                            <n-button type="primary" @click="unzipPlugin" :loading="uploading">
                                解压插件包
                            </n-button>

                            <n-button class="plugin-opt-but" type="primary" @click="downloadPlugin"
                                :loading="uploading">
                                远程下载
                            </n-button>

                            <n-button class="plugin-opt-but" @click="downloadPluginLog" :loading="uploading">
                                远程下载日志
                            </n-button>

                            <n-button class="plugin-opt-but" type="primary" @click="handleUpload"
                                :disabled="fileList.length === 0 || uploading" :loading="uploading"
                                style="margin-right: 8px;">
                                上传
                            </n-button>

                            <n-button class="plugin-opt-but" @click="fileList = []" :disabled="fileList.length === 0">
                                清空
                            </n-button>

                            <span>强制移动 <n-switch v-model:value="pluginRequest.force" /></span>
                            <n-button class="plugin-opt-but" type="warning" @click="movePlugin">
                                移动插件
                            </n-button>

                            <n-button class="plugin-opt-but" type="primary" @click="checkFilesConflict">
                                检查文件冲突
                            </n-button>
                        </n-alert>
                    </div>
                </div>

                <div style="margin-top: 16px">
                    <h4> 文件冲突 (Check files conflict plugin)</h4>
                    冲突文件列表:
                    <n-input :value="conflictList" type="textarea" size="medium"
                        style="width: 100%; min-height: 200px" />

                    完整数据:
                    <n-input :value="conflictInfo" type="textarea" size="medium" style="width: 100%; min-height: 200px"
                        readonly />
                </div>

            </n-card>

        </n-space>
    </div>
</template>

<script>
import { ref, computed } from 'vue';
import {
    NCard,
    NInput,
    NSelect,
    NButton,
    NSpace,
    NAlert,
    NDynamicTags,
    NEmpty,
    NIcon,
    useMessage
} from 'naive-ui';
import { Link as LinkIcon } from '@vicons/ionicons5';

export default {
    name: 'RequestAnalysisPage',
    components: {
        NCard,
        NInput,
        NSelect,
        NButton,
        NSpace,
        NAlert,
        NDynamicTags,
        NEmpty,
        NIcon,
        LinkIcon,
    },
    props: {
        analysisData: {
            type: Object,       // 假设你还要传递一个对象
            default: () => ({}), // 默认值为空对象
        },
    },
    setup(props) {
        const message = useMessage();
        // 请求相关
        const pluginRequestOrg = props?.analysisData || {
            filepath: '',
            downloadPluginUrl: ''
        };
        const pluginRequest = ref(pluginRequestOrg);

        // XXX 假数据
        pluginRequest.value.downloadPluginUrl = 'http://192.168.0.64:4433/pug.zip';

        const plugin = ref({});
        const loading = ref(false);

        // 格式化请求头显示（用于 DynamicTags 只读展示）
        const formatHeadersForDisplay = (headers) => {
            return Object.entries(headers).filter(([k, v]) => !['body', 'exclude'].includes(k)).map(([k, v]) => `${k}: ${v}`);
        };
        const simpleLsSetStack = ref(true);

        // 处理用户输入的 header（key:value）
        const handleHeaderCreate = (tag) => {
            // 可以做校验，这里直接返回 tag
            return tag;
        };

        // 查看详情
        const showPlugin = async (filepath) => {
            // 弹窗显示具体信息
            try {
                // 调用API接口
                const response = await fetch('/plugin-mgmt/api/plugin', {
                    method: 'POST',                       // 指定请求方法为 POST
                    headers: {
                        'Content-Type': 'application/json', // 告诉服务器发送的是 JSON
                    },
                    body: JSON.stringify({ filepath })
                });
                if (!response.ok) {
                    throw new Error(`HTTP error! status: ${response.status}`);
                }
                const result = await response.json();
                plugin.value = result.data
                message.success('数据获取成功');
            } catch (error) {
                console.error('获取数据失败:', error);
                message.error('数据加载失败，请重试(' + error.message + ')');
            } finally {
            }
        };

        const fileList = ref([])
        const uploading = ref(false)

        async function handleUpload() {
            // 上传文件
            if (fileList.value.length === 0) {
                message.warning('请先选择文件')
                return
            }

            const file = fileList.value[0]
            if (!file.name.toLowerCase().endsWith('.zip')) {
                message.error('只能上传 ZIP 文件')
                return
            }

            uploading.value = true

            try {
                // 这里替换为你的实际上传逻辑
                // 示例：创建 FormData 并发送请求
                const formData = new FormData()
                formData.append('uploadPlugin', file.file)

                const response = await fetch('/plugin-mgmt/api/upload', {
                    method: 'POST',
                    body: formData
                })

                if (!response.ok) {
                    throw new Error('上传失败')
                }

                // 假设成功响应包含插件内容，根据实际 API 响应结构调整
                const result = await response.json()

                console.info('上传成功', result);
                fileList.value = []
                message.success('上传成功')

            } catch (error) {
                console.error('上传失败', error);
                message.error('上传失败' + error.message);
            }
            uploading.value = false;
        }

        // 插件下载的日志信息
        const downloadPluginLog = async () => {
            try {
                const response = await fetch('/plugin-mgmt/api/downloadLog');
                if (!response.ok) {
                    throw new Error(`HTTP error! status: ${response.status}`);
                }
                const result = await response.json();
                message.success('下载日志: ' + result.data);
            } catch (error) {
                console.error('获取下载日志失败:', error);
                message.error('获取下载日志失败，请重试(' + error.message + ')');
            } finally {
            }
        };

        // 下载插件
        const downloadPlugin = async () => {
            // 弹窗显示具体信息
            try {
                // 调用API接口
                const response = await fetch('/plugin-mgmt/api/download', {
                    method: 'POST',                       // 指定请求方法为 POST
                    headers: {
                        'Content-Type': 'application/json', // 告诉服务器发送的是 JSON
                    },
                    body: JSON.stringify({ pluginUrl: pluginRequest.value.downloadPluginUrl })
                });
                if (!response.ok) {
                    throw new Error(`HTTP error! status: ${response.status}`);
                }
                const result = await response.json();
                console.log('下载结果 result.data', result.data);
                message.success('下载中');
            } catch (error) {
                console.error('下载失败:', error);
                message.error('下载失败，请重试(' + error.message + ')');
            } finally {
            }
        };

        // 插件下载的日志信息
        const unzipPlugin = async () => {
            try {
                message.info('解压中...');
                const response = await fetch('/plugin-mgmt/api/unzip', {
                    method: 'POST',                       // 指定请求方法为 POST
                    headers: {
                        'Content-Type': 'application/json', // 告诉服务器发送的是 JSON
                    },
                    body: JSON.stringify({
                        nameEncoding: pluginRequest.value.nameEncoding,
                        zipFilePath: pluginRequest.value.zipFilePath,
                    })
                });
                if (!response.ok) {
                    throw new Error(`HTTP error! status: ${response.status}`);
                }
                const result = await response.json();
                console.log('解压结果: ' + result.data);
                message.success('解压结果: ' + JSON.stringify(result.data));
            } catch (error) {
                console.error('解压结果失败:', error);
                message.error('解压结果失败，请重试(' + error.message + ')');
            } finally {
            }
        };

        const conflict = ref({
            conflicts: []
        });
        const conflictInfo = computed(() => {
            const text = JSON.stringify(conflict.value, null, 2)
            return text
        });
        const conflictList = computed(() => {
            try {
                return conflict.value.conflicts.join('\n') || '无'
            } catch (err) {
                return err.message
            }
        });

        // 下载插件
        const checkFilesConflict = async () => {
            // 弹窗显示具体信息
            try {
                message.info('比对文件中...');
                // 调用API接口
                const response = await fetch('/plugin-mgmt/api/checkFilesConflict', {
                    method: 'POST',                       // 指定请求方法为 POST
                    headers: {
                        'Content-Type': 'application/json', // 告诉服务器发送的是 JSON
                    },
                    body: JSON.stringify({
                        fromPath: pluginRequest.value.fromPath,
                        toPath: pluginRequest.value.toPath,
                    })
                });
                if (!response.ok) {
                    throw new Error(`HTTP error! status: ${response.status}`);
                }
                const result = await response.json();
                console.log('对比结果', result.data);
                conflict.value = result.data
                message.success('对比完成: ' + result.data.conflicts.length + '条冲突数据');
            } catch (error) {
                console.error('对比失败:', error);
                message.error('对比失败，请重试(' + error.message + ')');
            } finally {
            }
        };

        // 移动插件
        const movePlugin = async () => {
            // 弹窗显示具体信息
            try {
                message.info('移动文件中...');
                // 调用API接口
                const response = await fetch('/plugin-mgmt/api/movePlugin', {
                    method: 'POST',                       // 指定请求方法为 POST
                    headers: {
                        'Content-Type': 'application/json', // 告诉服务器发送的是 JSON
                    },
                    body: JSON.stringify({
                        force: pluginRequest.value.force,
                    })
                });
                if (!response.ok) {
                    throw new Error(`HTTP error! status: ${response.status}`);
                }
                const result = await response.json();
                console.log('移动结果', result.data);
                conflict.value = result.data
                if (result.code != 200) throw new Error(result.code + ': ' + result.msg)
                message.success('移动完成: ' + result.data.fromFiles.length);
            } catch (error) {
                console.error('移动失败:', error);
                message.error('移动失败，请重试(' + error.message + ')');
            } finally {
            }
        };

        return {
            plugin,
            pluginRequest,
            loading,
            formatHeadersForDisplay,
            handleHeaderCreate,
            simpleLsSetStack,
            showPlugin,
            handleUpload,
            fileList,
            uploading,
            downloadPlugin,
            downloadPluginLog,
            unzipPlugin,
            checkFilesConflict,
            conflict,
            conflictInfo,
            conflictList,
            movePlugin,
        };
    },
};
</script>

<style scoped>
.request-analysis-page {
    padding: 16px;
}

.plugin-opt-but {
    margin-left: 10px;
}
</style>