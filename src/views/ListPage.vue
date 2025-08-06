<template>
  <div class="table-container">
    <!-- 使用标签页切换两个页面 -->
    <n-tabs v-model:value="activeTab">
      <!-- 标签1：插件列表 -->
      <n-tab-pane name="list" tab="插件列表">
        <!-- 操作按钮区域 -->
        <div class="table-actions">
          <n-button type="primary" @click="fetchData" icon-placement="left" style="margin-left:0">
            刷新数据 ({{ data.length }})
          </n-button>

          <n-button type="primary" strong secondary @click="scrollToBottom" icon-placement="left">
            前往底部
          </n-button>
          <n-button type="primary" @click="switchPlugin()" icon-placement="left">
            全部反向启用/禁用
          </n-button>
        </div>
        <n-data-table :columns="columns" :data="data" :pagination="pagination" :bordered="false" :loading="loading" />
      </n-tab-pane>

      <!-- 标签2：请求分析页面 -->
      <n-tab-pane name="analysis" tab="其他功能">
        <!-- 这里放你的“请求分析”内容组件 -->
        <div style="width: 90vw;">
          <RequestAnalysisPage :analysisData="analysisData"></RequestAnalysisPage>
        </div>
      </n-tab-pane>
    </n-tabs>
  </div>
</template>

<script>
import { NButton, useMessage, NTag, useDialog } from "naive-ui";
import { defineComponent, h, ref, computed, onMounted } from "vue";
import RequestAnalysisPage from './RequestAnalysisPage.vue'; // 你要创建的分析页面组件

// 创建表格列定义
function createColumns({ play, showPlugin, switchPlugin }) {
  return [
    {
      title: "No",
      key: "no",
      width: 50,
      fixed: "left",
      render(row, index) {
        return index + 1;
      }
    },
    {
      title: "阶段 (Stage)",
      key: "stage",
      width: 150,
    },
    {
      title: "名称 (Filename)",
      key: "filename",
      width: 200,
      render(row) {
        return h(
          "span",
          { style: { color: row.exclude ? "red" : "" } },
          row.filename // 显示原始文件名
        );
      }
    },
    {
      title: "目录 (Dirname)",
      key: "dirname",
      width: 300,
    },
    {
      title: "操作",
      key: "actions",
      width: 120,
      fixed: "right",
      render(row) {
        return h(
          "div", // 使用 div 包裹按钮组
          { style: "display: flex; gap: 8px;" }, // 添加间距
          [
            // 原详情按钮
            h(
              NButton,
              {
                strong: true,
                tertiary: true,
                size: "small",
                onClick: () => showPlugin ? showPlugin(row) : null
              },
              { default: () => "详情" }
            ),
            // 新增切换状态按钮
            h(
              NButton,
              {
                strong: true,
                type: row.exclude ? "primary" : "warning", // 可选，用于区分按钮类型
                size: "small",
                onClick: () => switchPlugin ? switchPlugin(row) : null
              },
              { default: () => row.exclude ? "启用" : "禁用" }
            )
          ]
        );
      }
    }
  ];
}

export default defineComponent({
  components: {
    RequestAnalysisPage,
  },
  setup() {
    const message = useMessage();
    const dialog = useDialog()
    // 状态管理
    /** @type {[{"name":null,"path":"/demo","regexp":"reg:/^\\/demo[\\/#\\?]?$/i","opts":{"end":true,"name":null,"sensitive":false,"strict":false,"prefix":""},"paramNames":[],"methods":["HEAD","ACL","BIND","CHECKOUT","CONNECT","COPY","DELETE","GET","HEAD","LINK","LOCK","M-SEARCH","MERGE","MKACTIVITY","MKCALENDAR","MKCOL","MOVE","NOTIFY","OPTIONS","PATCH","POST","PROPFIND","PROPPATCH","PURGE","PUT","QUERY","REBIND","REPORT","SEARCH","SOURCE","SUBSCRIBE","TRACE","UNBIND","UNLINK","UNLOCK","UNSUBSCRIBE"],"remark":"动态路由"}]} */
    const data = ref([]);   // 全部数据
    const loading = ref(false); // 加载状态
    const analysisData = ref({
      url: '',
      method: 'GET',
      headers: [], // 格式：["Content-Type:application/json", "Authorization:Bearer xxx"]
      body: '',
    });

    // 分页配置
    const paginationReactive = ref({
      page: 1,
      pageSize: 10,
      showSizePicker: true,
      pageSizes: [10, 30, 50, 100, 300, 500, 1000, 5000, 10000, 300000, 1000000],
      onChange: (page) => {
        paginationReactive.value.page = page
      },
      onUpdatePageSize: (pageSize) => {
        if (pageSize != undefined) paginationReactive.value.pageSize = pageSize;
        paginationReactive.value.page = 1
      }
    });

    // 默认选中 "数据列表" 标签
    const activeTab = ref('list');

    /**
     * 
     * @param data {{ "stages": ["systemStart", "koaPlugin", "koaRouter", "selectFileByDomains", "genProxy", "indexData", "websocketMsgs", "websocketApis", "cmd", "send"], "pluginPath": ["/root/Project/LazierServer/ld/plugins/indexData-1-demo.js", "/root/Project/LazierServer/ld/plugins/koaPlugin-1-demo.js", "/root/Project/LazierServer/ld/plugins/koaPlugin-2-cors.js", "/root/Project/LazierServer/ld/plugins/koaPlugin-3-bodyparser.js", "/root/Project/LazierServer/ld/plugins/koaPlugin-4-counter.js", "/root/Project/LazierServer/ld/plugins/koaPlugin-5-sw.js302Fix.js", "/root/Project/LazierServer/ld/plugins/koaPlugin-6-rewritePathnameToIndex.html.js", "/root/Project/LazierServer/ld/plugins/koaPlugin-7-proxies.js", "/root/Project/LazierServer/ld/plugins/koaRouter-1-demo.js", "/root/Project/LazierServer/ld/plugins/koaRouter-1.1-pluginManagement.js", "/root/Project/LazierServer/ld/plugins/koaRouter-2-scanWeb.js", "/root/Project/LazierServer/ld/plugins/koaRouter-3-scanHar.js", "/root/Project/LazierServer/ld/plugins/koaRouter-4-system.js", "/root/Project/LazierServer/ld/plugins/koaRouter-5-history.js", "/root/Project/LazierServer/ld/plugins/selectFileByDomains-demo.js", "/root/Project/LazierServer/ld/plugins/send-redirectApi.js", "/root/Project/LazierServer/ld/plugins/send-sameApi.js", "/root/Project/LazierServer/ld/plugins/send-system.js", "/root/Project/LazierServer/ld/plugins/systemStart-addAppStack.js", "/root/Project/LazierServer/ld/plugins/systemStart-common.js", "/root/Project/LazierServer/ld/plugins/systemStart-logger.js", "/root/Project/LazierServer/ld/plugins/websocketApis-demo.js", "/root/Project/LazierServer/ld/plugins/websocketMsgs-demo.js"], "excludePlugins": ["/root/Project/LazierServer/ld/plugins/indexData-1-demo.js", "/root/Project/LazierServer/ld/plugins/koaPlugin-1-demo.js"] }}
     */
    const handlerData = (data) => {
      const re = [];
      const stages = data.stages;
      const excludePlugins = data.excludePlugins;
      data.pluginPath.forEach(filepath => {
        const filename = filepath.split('/').pop().split('\\').pop();
        const dirname = filepath.replace(filename, '');
        const stage = stages.find(s => filename.startsWith(s));
        re.push({
          filepath,
          filename,
          dirname,
          stage,
          exclude: excludePlugins.includes(filepath)
        });
      })
      return re;
    }
    // 从API获取数据
    const fetchData = async () => {
      try {
        loading.value = true;
        // 调用API接口
        const response = await fetch('/plugin-mgmt/api/pluginList');

        if (!response.ok) {
          throw new Error(`HTTP error! status: ${response.status}`);
        }

        const result = await response.json();
        data.value = handlerData(result.data);
        message.success('数据加载成功');
        paginationReactive.value.onUpdatePageSize();
      } catch (error) {
        console.error('获取数据失败:', error);
        message.error('数据加载失败，请重试');
      } finally {
        loading.value = false;
      }
    };

    // 播放操作
    const play = (row) => {
      analysisData.value.filepath = row.filepath;
      message.info(`已选择 ${row.filepath}`);
      activeTab.value = 'analysis';
    };
    const showPlugin = (row) => {
      analysisData.value.filepath = row.filepath;
      message.info(`已选择 ${row.filepath}`);
      activeTab.value = 'analysis';
    };

    // 查看详情
    const switchPlugin = async (row) => {
      // 弹窗显示具体信息
      try {
        const filepathList = [];
        const selectList = [];
        if (row) {
          filepathList.push(row.filepath);
          selectList.push(row);
        } else {
          // 全选
          const page = paginationReactive.value;
          const max = page.page * page.pageSize;
          selectList = data.value.slice(max - page.pageSize, max);
          selectList.forEach(e => {
            filepathList.push(e.filepath)
          });
        }

        // 调用API接口
        const response = await fetch('/plugin-mgmt/api/switch', {
          method: 'POST',                    // 指定请求方法为 POST
          headers: {
            'Content-Type': 'application/json', // 告诉服务器发送的是 JSON
          },
          body: JSON.stringify({ filepathList })
        });
        if (!response.ok) {
          throw new Error(`HTTP error! status: ${response.status}`);
        }
        const result = await response.json();
        // 只更新当前的列表
        selectList.forEach(r => r.exclude = !r.exclude);
      } catch (error) {
        console.error('获取数据失败:', error);
        message.error('数据加载失败，请重试');
      } finally {
      }
    };

    // 组件挂载时获取数据
    onMounted(() => {
      fetchData();
    });

    const scrollToBottom = () => {
      window.scrollTo({
        top: document.documentElement.scrollHeight,
        behavior: 'smooth' // 平滑滚动，可去掉
      });
    };

    return {
      data,
      columns: createColumns({ play, showPlugin, switchPlugin }),
      pagination: paginationReactive,
      loading,
      fetchData,
      scrollToBottom,
      activeTab,
      analysisData,
      switchPlugin
    };
  }
});

</script>

<style scoped>
.table-container {
  padding: 16px;
}

.table-actions {
  margin-bottom: 16px;
  text-align: left;
}

.table-actions>* {
  margin-left: 16px;
}

.n-tabs-nav--bar-type.n-tabs-nav--top.n-tabs-nav {
  position: sticky;
  top: 0;
}
</style>

<style>
/* 默认：只显示一行，超出隐藏 */
.http-methods.collapsed {
  white-space: nowrap;
  overflow: hidden;
  display: flex;
  flex-wrap: nowrap;
  /* 可选：限制为单行标签的大致高度 */
  height: 24px;
}

/* 展开状态：允许换行，显示全部 */
.http-methods.expanded {
  white-space: normal;
  overflow: visible;
  display: flex;
  flex-wrap: wrap;
}

.http-methods>.n-tag {
  font-size: 60%;
}
</style>
