<template>
    <div class="request-analysis-page">
        <n-space vertical size="large" style="width: 100%">
            <!-- ===== 其他功能 ===== -- >
            <n-card title="📤 其他功能" size="small">
                <n-space vertical size="medium">
                    <n-input v-model:value="pluginRequest.filepath" placeholder="请输入请求 FilePath (支持 http / https)"
                        size="medium" style="width: 100%">
                        <template #prefix>
                            <n-icon>
                                <LinkIcon />
                            </n-icon>
                        </template>
</n-input>

<n-button type="primary" size="medium" @click="showPlugin(pluginRequest.filepath)" :loading="loading" block>
    远程下载
</n-button>
</n-space>
</n-card>
-->

            <!-- ===== 请求配置区域 ===== -->
            <n-card title="📤 插件地址" size="small">
                <n-space vertical size="medium">
                    <!-- FilePath -->
                    <n-input v-model:value="pluginRequest.filepath" placeholder="请输入请求 FilePath (支持 http / https)"
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
            </n-card>

            <!-- ===== 响应展示区域 ===== -->
            <n-card title="📥 插件信息" size="small">
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
        </n-space>
    </div>
</template>

<script>
import { ref } from 'vue';
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
            filepath: ''
        };
        const pluginRequest = ref(pluginRequestOrg);
        const plugin = ref({});
        const loading = ref(false);

        // 格式化请求头显示（用于 DynamicTags 只读展示）
        const formatHeadersForDisplay = (headers) => {
            return Object.entries(headers).filter(([k, v]) => !['body', 'exclude'].includes(k)).map(([k, v]) => `${k}:${v}`);
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
                message.error('数据加载失败，请重试');
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
            showPlugin
        };
    },
};
</script>

<style scoped>
.request-analysis-page {
    padding: 16px;
}
</style>