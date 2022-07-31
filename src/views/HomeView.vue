<script lang="ts" setup>
import { computed, onMounted, ref } from "vue";

//-----------页面布局
const styleHeaderHeight = "60px";

interface IPanelWidth {
  left: number;
  main: number;
  right: number;
}

const panelWidth = ref<IPanelWidth>({
  left: 20,
  main: 60,
  right: 20,
});
const stylePanelWidth = computed(() => {
  return new Proxy(panelWidth.value, {
    get: (target, property: keyof IPanelWidth, receiver) => {
      return `${target[property]}%`;
    },
  });
});

//----------左侧组件栏
import * as monaco from "monaco-editor";
import editorWorker from "monaco-editor/esm/vs/editor/editor.worker?worker";
import jsonWorker from "monaco-editor/esm/vs/language/json/json.worker?worker";
// import cssWorker from "monaco-editor/esm/vs/language/css/css.worker?worker";
// import htmlWorker from "monaco-editor/esm/vs/language/html/html.worker?worker";
// import tsWorker from "monaco-editor/esm/vs/language/typescript/ts.worker?worker";

// eslint-disable-next-line @typescript-eslint/ban-ts-comment
// @ts-ignore
window.MonacoEnvironment = {
  getWorker(_: string, label: string) {
    if (label === "json") {
      return new jsonWorker();
    }
    // if (label === "css" || label === "scss" || label === "less") {
    //   return new cssWorker();
    // }
    // if (label === "html" || label === "handlebars" || label === "razor") {
    //   return new htmlWorker();
    // }
    // if (label === "typescript" || label === "javascript") {
    //   return new tsWorker();
    // }
    return new editorWorker();
  },
};
import { useStorage } from "@/stores/storage";
import { storeToRefs } from "pinia";

const Storage = useStorage();
const { code } = storeToRefs(Storage);
const codeContainer = ref<InstanceType<typeof HTMLElement>>();
onMounted(() => {
  monaco.editor.create(codeContainer.value!, {
    value: JSON.stringify(code.value),
    language: "json",
    automaticLayout: true, //开启自适应大小
    minimap: {
      enabled: false, // 不需要小的缩略图
    },
    lineNumbers: "off", //关闭显示行号
    folding: false, //关闭折叠
  });
});

//-------------拖拽
const DRAG_DATA_CONSTANTS = {
  dataTransferKey: "componentName",
};

function onPhoneDragEnter() {}

function onPhoneDragLeave() {}

function onPhoneDragDrop(e: DragEvent) {
  console.log(e.dataTransfer!.getData(DRAG_DATA_CONSTANTS.dataTransferKey));
}

function onLibraryComponentDragStart(e: DragEvent) {
  e.dataTransfer!.setData(DRAG_DATA_CONSTANTS.dataTransferKey, "test");
}
</script>

<template>
  <el-container class="app-container h-screen">
    <!--    顶栏-->
    <el-header :height="styleHeaderHeight" class="shadow">
      <el-row
        :style="{ height: styleHeaderHeight, 'line-height': styleHeaderHeight }"
      >
        <el-col :span="4"> LowCodeDemo</el-col>
        <el-col :span="16"></el-col>
        <el-col :span="4" class="justify-end flex items-center">
          <el-button type="danger">重置</el-button>
          <el-button type="primary">预览</el-button>
        </el-col>
      </el-row>
    </el-header>

    <!--    下部中间操作区-->
    <el-main>
      <div class="main-wrapper h-full">
        <!--        左侧面板-->
        <div class="panel panel-left h-full">
          <el-tabs tab-position="left" type="border-card">
            <el-tab-pane label="通用组件">
              <el-collapse>
                <!--                表单组件-->
                <el-collapse-item title="表单">
                  <el-row>
                    <el-col
                      v-for="(e, index) in new Array(10)"
                      :key="index"
                      :span="12"
                    >
                      <div
                        class="button library-component"
                        draggable="true"
                        @dragstart="onLibraryComponentDragStart"
                      >
                        <div class="component-icon">
                          <el-icon :size="16">
                            <Aim />
                          </el-icon>
                        </div>
                        <div class="desc">按钮</div>
                        <div class="ask-icon">
                          <el-tooltip
                            effect="light"
                            placement="right"
                            popper-class="tips-wrapper"
                          >
                            <template #default>
                              <el-icon :size="16">
                                <QuestionFilled />
                              </el-icon>
                            </template>
                            <template #content>
                              <div class="tips-panel">
                                <div class="tips-title">按钮</div>
                                <div class="tips-desc">
                                  用来展示一个按钮，你可以配置不同的展示样式，配置不同的点击行为。
                                </div>
                                <div class="tips-preview">
                                  <el-button>Default</el-button>
                                  <el-button type="primary">Primary</el-button>
                                  <el-button type="success">Success</el-button>
                                </div>
                              </div>
                            </template>
                          </el-tooltip>
                        </div>
                      </div>
                    </el-col>
                  </el-row>
                </el-collapse-item>
                <!--                按钮组件-->
                <el-collapse-item title="按钮"> 333</el-collapse-item>
                <!--                展示组件-->
                <el-collapse-item title="展示"> 444</el-collapse-item>
              </el-collapse>
            </el-tab-pane>
            <!--            代码面板-->
            <el-tab-pane label="代码">
              <div
                ref="codeContainer"
                class="code-container h-full w-full"
              ></div>
            </el-tab-pane>
          </el-tabs>
        </div>

        <!--        中间手机模型-->
        <div class="panel panel-main h-full">
          <div class="phone-wrapper">
            <div
              class="phone"
              @dragenter="onPhoneDragEnter"
              @dragleave="onPhoneDragLeave"
              @dragover="(e:DragEvent)=>{e.preventDefault()}"
              @drop="onPhoneDragDrop"
            ></div>
          </div>
        </div>

        <!--        右侧参数面板-->
        <div class="panel panel-right h-full">
          <el-tabs type="border-card">
            <el-tab-pane label="常规"> 1212</el-tab-pane>
            <el-tab-pane label="外观"> 1212</el-tab-pane>
            <el-tab-pane label="其他">1321</el-tab-pane>
          </el-tabs>
        </div>
      </div>
    </el-main>
  </el-container>
</template>

<style lang="scss">
//组件使用提示
.tips-wrapper {
  @apply shadow;
}
</style>

<style lang="scss" scoped>
.app-container {
  :deep(.el-main) {
    --el-main-padding: 0;
  }

  :deep(.el-collapse-item__header) {
    --el-collapse-header-height: 35px;
  }
}

//组件使用提示
.tips-panel {
  max-width: 328px;
  min-height: 80px;
}

.main-wrapper {
  @apply flex;
  .panel {
    @apply flex;
  }

  .panel-left {
    width: max(v-bind("stylePanelWidth.left"), 385px);

    .el-tabs {
      @apply flex-grow;
      :deep(.el-tabs__content) {
        @apply h-full;
        .el-tab-pane {
          @apply h-full;
        }
      }
    }

    .el-collapse {
      @apply border-transparent #{!important};
      :deep(.el-collapse-item__wrap),
      :deep(.el-collapse-item__header) {
        @apply border-transparent #{!important};
      }

      :deep(.el-collapse-item__content) {
        padding-bottom: 0;
      }
    }

    .library-component {
      @apply flex items-center border-solid rounded border cursor-pointer truncate;
      border-color: #e8e9eb;
      padding: 0 8px;
      width: 112px;
      height: 32px;
      margin-bottom: 12px;

      &:hover {
        border-color: #2468f2;
      }

      .desc {
        @apply flex-auto;
        padding: 0 8px;
      }

      .ask-icon,
      .component-icon {
        @apply flex justify-center items-center;
        flex: 0 0 16px;
      }

      .component-icon {
        width: 16px;
        height: 16px;
      }
    }
  }

  .panel-main {
    @apply flex-1;

    .phone-wrapper {
      flex-basis: 375px;
      min-height: 812px;

      @apply flex-col flex m-auto;
      .phone {
        box-shadow: 0 0 20px 0 rgba(0, 0, 0, 0.2);
        @apply h-full w-full flex-1;
      }
    }
  }

  .panel-right {
    width: max(v-bind("stylePanelWidth.left"), 385px);

    .el-tabs {
      @apply flex-grow flex-col;
    }
  }
}
</style>
