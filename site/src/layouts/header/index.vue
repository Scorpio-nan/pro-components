<template>
  <header id="header" :class="headerClassName">
    <!-- <div v-if="visibleAdblockBanner" class="adblock-banner">
      <template v-if="isZhCN">
        我们检测到你可能使用了 AdBlock 或 Adblock
        Plus，它会影响到正常功能的使用（如复制、展开代码等）。
        <br />
        你可以将 ProComponents 加入白名单，以便我们更好地提供服务。
      </template>
      <template v-else>
        We have detected that you may use AdBlock or Adblock Plus, which will affect the use of
        normal functions (such as copying, expanding code, etc.)
        <br />
        You can add ProComponents to the whitelist so that we can provide better services.
      </template>

      <CloseOutlined class="close-icon" @click="visibleAdblockBanner = false" />
    </div> -->
    <!-- <div v-if="visibleAlertBanner && isZhCN" class="alert-banner">
      Surely Form 2.0 发布，快速定制自己的问卷平台 &nbsp;&nbsp;
      <a href="https://form.antdv.com">立即体验</a>

      <CloseOutlined class="close-icon" @click="visibleAlertBanner = false" />
    </div> -->
    <!-- <a-popover
      v-model:visible="menuVisible"
      overlay-class-name="popover-menu"
      placement="bottomRight"
      trigger="click"
      arrow-point-at-center
    >
      <UnorderedListOutlined class="nav-phone-icon" />
      <template #content>
        <Menu :is-mobile="isMobile" />
      </template>
    </a-popover> -->
    <a-row :style="{ flexFlow: 'nowrap', height: 64 }">
      <a-col v-bind="colProps[0]">
        <Logo />
      </a-col>
      <a-col v-bind="colProps[1]" class="menu-row">
        <!-- <SearchBox
          key="search"
          :is-zh-c-n="isZhCN"
          :responsive="responsive"
          @triggerFocus="onTriggerSearching"
        /> -->
        <!-- TODO 占位, 搜索功能之后接入 -->
        <div id="search-box"></div>
        <Menu v-if="!isMobile" />
      </a-col>
    </a-row>
  </header>
</template>
<script lang="ts">
import type { GlobalConfig } from '../../App.vue';
import { GLOBAL_CONFIG } from '../../SymbolKey';
import { getLocalizedPathname } from '../../utils/util';
import { computed, defineComponent, inject, onMounted, ref, watch } from 'vue';
import { useRoute } from 'vue-router';
import Logo from './Logo.vue';
import Menu from './Menu.vue';
import { UnorderedListOutlined, CloseOutlined } from '@ant-design/icons-vue';
import SearchBox from './SearchBox.vue';
import { version } from 'ant-design-vue';
export default defineComponent({
  components: {
    Logo,
    Menu,
    UnorderedListOutlined,
    SearchBox,
    CloseOutlined,
  },
  setup() {
    const route = useRoute();
    const globalConfig = inject<GlobalConfig>(GLOBAL_CONFIG);
    const isHome = computed(() => {
      return ['', 'index', 'index-cn'].includes(route.path);
    });

    const menuVisible = ref(false);
    const colProps = isHome.value
      ? [{ flex: 'none' }, { flex: 'auto' }]
      : [
          {
            xxxl: 4,
            xxl: 4,
            xl: 5,
            lg: 6,
            md: 6,
            sm: 24,
            xs: 24,
          },
          {
            xxxl: 20,
            xxl: 20,
            xl: 19,
            lg: 18,
            md: 18,
            sm: 0,
            xs: 0,
          },
        ];
    const searching = ref(false);
    const onTriggerSearching = (value: boolean) => {
      searching.value = value;
    };
    const initDocSearch = () => {
      const searchBox = document.querySelector('#search-box');
      if (!searchBox || searchBox.tagName === "DIV") { return; }
      window.docsearch({
        apiKey: '92003c1d1d07beef165b08446f4224a3',
        indexName: 'antdv',
        inputSelector: '#search-box input',
        algoliaOptions: { facetFilters: [`tags:${globalConfig?.isZhCN.value ? 'cn' : 'en'}`] },
        transformData(hits: any[]) {
          hits.forEach(hit => {
            hit.url = hit.url.replace('www.antdv.com', window.location.host);
            hit.url = hit.url.replace('https:', window.location.protocol);
          });
          return hits;
        },
        debug: false, // Set debug to true if you want to inspect the dropdown
      });
    };
    onMounted(() => {
      setTimeout(() => {
        initDocSearch();
      });
    });
    const visibleAdblockBanner = ref(false);
    watch(globalConfig?.blocked, val => {
      visibleAdblockBanner.value = val;
    });
    const visibleAlertBanner = ref(!localStorage.getItem('surelyform_v2'));
    watch(visibleAlertBanner, () => {
      if (!visibleAlertBanner.value) {
        localStorage.setItem('surelyform_v2', version);
      }
    });
    return {
      isZhCN: globalConfig.isZhCN,
      isMobile: globalConfig.isMobile,
      responsive: globalConfig.responsive,
      getLocalizedPathname,
      visibleAdblockBanner,
      headerClassName: {
        clearfix: true,
        'home-header': isHome.value,
      },
      colProps,
      menuVisible,
      onTriggerSearching,
      visibleAlertBanner,
    };
  },
});
</script>
<style lang="less" src="./index.less"></style>
<style scope>
.adblock-banner,
.alert-banner {
  position: relative;
  z-index: 100;
  padding: 16px;
  line-height: 28px;
  color: #8590a6;
  text-align: center;
  background-color: #ebebeb;
}
.alert-banner {
  background-color: var(--ant-primary-color);
  color: #fff;
  padding: 5px;
}
.alert-banner a {
  color: #fff;
  text-decoration: underline;
}
.alert-banner .close-icon {
  top: 12px;
}
.close-icon {
  position: absolute;
  top: 15px;
  right: 15px;
}
</style>
