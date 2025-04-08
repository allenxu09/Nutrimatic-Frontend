<template>
  <div class="min-h-screen bg-gray-50 flex flex-col">
    <!-- 顶部导航栏 -->
    <nav class="bg-blue-700 text-white shadow-md">
      <div class="container mx-auto px-4 py-3 flex items-center justify-between">
        <h1 class="text-2xl font-bold">成语查询</h1>
        <button
          @click="getRandomIdiom"
          class="bg-blue-600 hover:bg-blue-800 px-4 py-2 rounded-lg transition">
          随机成语
        </button>
      </div>
    </nav>

    <!-- 主要内容区 -->
    <div class="container mx-auto px-4 py-8 flex-grow">
      <!-- 搜索区域 -->
      <div class="bg-white rounded-xl shadow-lg p-6 mb-8">
        <h2 class="text-xl font-semibold mb-4 text-gray-800">快速查询成语</h2>

        <form @submit.prevent="search" class="space-y-4">
          <div class="flex flex-col md:flex-row gap-4">
            <div class="flex-grow">
              <input
                v-model="searchParams.query"
                type="text"
                placeholder="请输入查询内容..."
                class="w-full px-4 py-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500 outline-none"
                required
              />
            </div>

            <button
              type="submit"
              class="bg-blue-600 hover:bg-blue-700 text-white font-medium px-6 py-3 rounded-lg transition flex items-center justify-center"
              :disabled="loading"
            >
              <span v-if="loading" class="mr-2">
                <svg class="animate-spin h-5 w-5" viewBox="0 0 24 24">
                  <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4" fill="none"></circle>
                  <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
                </svg>
              </span>
              <span>搜索</span>
            </button>
          </div>

          <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
            <div>
              <label class="block text-sm font-medium text-gray-700 mb-1">搜索类型</label>
              <select
                v-model="searchParams.search_type"
                class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-blue-500 focus:border-blue-500"
              >
                <option value="pinyin">拼音搜索</option>
                <option value="word">词语搜索</option>
                <option value="first_char">首字符搜索</option>
                <option value="explanation">释义搜索</option>
                <option value="regex">正则表达式</option>
              </select>
            </div>

            <div class="flex items-center">
              <label class="inline-flex items-center">
                <input type="checkbox" v-model="searchParams.exact_match" class="h-5 w-5 text-blue-600">
                <span class="ml-2 text-gray-700">精确匹配</span>
              </label>

              <div class="ml-6">
                <label class="inline-flex items-center">
                  <span class="text-gray-700 mr-2">每页显示</span>
                  <select
                      v-model="searchParams.limit"
                      class="border border-gray-300 rounded px-2 py-1 focus:ring-blue-500 focus:border-blue-500"
                  >
                    <option :value="10">10</option>
                    <option :value="20">20</option>
                    <option :value="50">50</option>
                    <option :value="100">100</option>
                  </select>
                </label>
              </div>
            </div>
          </div>
        </form>
      </div>

      <!-- 错误信息 -->
      <div v-if="error" class="bg-red-100 border-l-4 border-red-500 text-red-700 p-4 mb-6 rounded">
        <p>{{ error }}</p>
      </div>

      <!-- 搜索结果 -->
      <div v-if="searchResult" class="bg-white rounded-xl shadow-lg p-6">
        <div class="flex justify-between items-center mb-4">
          <h3 class="text-lg font-semibold text-gray-800">
            搜索结果 (共{{ searchResult.total }}条)
          </h3>
          <div v-if="searchResult.total > searchParams.limit" class="flex items-center">
            <button
                @click="prevPage"
                :disabled="searchParams.offset === 0"
                class="px-3 py-1 border rounded-md mr-2 disabled:opacity-50"
            >
              上一页
            </button>
            <span class="text-gray-600">
              {{ Math.floor(searchParams.offset / searchParams.limit) + 1 }} /
              {{ Math.ceil(searchResult.total / searchParams.limit) }}
            </span>
            <button
                @click="nextPage"
                :disabled="searchParams.offset + searchParams.limit >= searchResult.total"
                class="px-3 py-1 border rounded-md ml-2 disabled:opacity-50"
            >
              下一页
            </button>
          </div>
        </div>

        <div v-if="searchResult.results.length === 0" class="text-center py-8 text-gray-500">
          未找到匹配的成语
        </div>

        <div v-else class="grid grid-cols-1 md:grid-cols-2 gap-4">
          <div
              v-for="(idiom, index) in searchResult.results"
              :key="index"
              class="border border-gray-200 rounded-lg p-4 hover:shadow-md transition cursor-pointer"
              @click="showIdiomDetail(idiom)"
          >
            <h4 class="text-xl font-medium text-gray-900">{{ idiom.word }}</h4>
            <p class="text-gray-500">{{ idiom.pinyin }}</p>
            <p class="text-gray-600 mt-1 line-clamp-2">{{ idiom.explanation }}</p>
          </div>
        </div>
      </div>
        <RegexHelper
          v-if="searchParams.search_type === 'regex'"
          @apply-pattern="applyRegexPattern"
          class="mt-5"
        />
    </div>

    <!-- 成语详情弹窗 -->
    <div
        v-if="showDetail"
        class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center p-4 z-50"
        @click="showDetail = false"
    >
      <div
          class="bg-white rounded-xl max-w-lg w-full p-6 max-h-[80vh] overflow-y-auto"
          @click.stop
      >
        <div class="flex justify-between items-start">
          <h3 class="text-2xl font-bold text-gray-900">{{ selectedIdiom.word }}</h3>
          <button @click="showDetail = false" class="text-gray-500 hover:text-gray-700">
            <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24"
                 stroke="currentColor">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"/>
            </svg>
          </button>
        </div>

        <div class="mt-4 space-y-3">
          <div>
            <span class="text-gray-500">拼音：</span>
            <span class="text-gray-900">{{ selectedIdiom.pinyin }}</span>
          </div>
          <div>
            <span class="text-gray-500">释义：</span>
            <p class="text-gray-900">{{ selectedIdiom.explanation }}</p>
          </div>
          <div v-if="selectedIdiom.derivation">
            <span class="text-gray-500">出处：</span>
            <p class="text-gray-900">{{ selectedIdiom.derivation }}</p>
          </div>
          <div v-if="selectedIdiom.example">
            <span class="text-gray-500">例句：</span>
            <p class="text-gray-900">{{ selectedIdiom.example }}</p>
          </div>
        </div>

        <div class="mt-6 flex justify-end">
          <button
              @click="copyIdiom(selectedIdiom.word)"
              class="bg-gray-100 hover:bg-gray-200 text-gray-800 px-4 py-2 rounded-lg transition mr-2"
          >
            复制成语
          </button>
          <button
              @click="showDetail = false"
              class="bg-blue-600 hover:bg-blue-700 text-white px-4 py-2 rounded-lg transition"
          >
            关闭
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import RegexHelper from "./components/RegexHelper.vue";
export default {
  components:{
    RegexHelper
  },
  data() {
    return {
      searchParams: {
        query: "",
        search_type: "pinyin",
        exact_match: false,
        limit: 20,
        offset: 0
      },
      searchResult: null,
      loading: false,
      error: "",
      showDetail: false,
      selectedIdiom: {},
      apiBaseUrl: "http://localhost:8000"
    };
  },
  methods: {
    applyRegexPattern(pattern) {
      this.searchParams.query = pattern;
      this.search(); // 自动执行搜索
    },
    async search() {
      this.error = "";
      this.loading = true;

      try {
        const response = await axios.post(
            `${this.apiBaseUrl}/search`,
            this.searchParams
        );
        this.searchResult = response.data;
      } catch (err) {
        this.error = err.response?.data?.detail || "搜索过程中发生错误";
        console.error(err);
      } finally {
        this.loading = false;
      }
    },

    prevPage() {
      if (this.searchParams.offset >= this.searchParams.limit) {
        this.searchParams.offset -= this.searchParams.limit;
        this.search();
      }
    },

    nextPage() {
      if (this.searchParams.offset + this.searchParams.limit < this.searchResult.total) {
        this.searchParams.offset += this.searchParams.limit;
        this.search();
      }
    },

    async showIdiomDetail(idiom) {
      try {
        // 如果对象中已有完整信息，直接使用
        if (idiom.explanation) {
          this.selectedIdiom = idiom;
          this.showDetail = true;
          return;
        }

        // 否则请求详细信息
        const response = await axios.get(`${this.apiBaseUrl}/idiom/${idiom.word}`);
        this.selectedIdiom = response.data;
        this.showDetail = true;
      } catch (err) {
        this.error = "获取成语详情失败";
        console.error(err);
      }
    },

    async getRandomIdiom() {
      try {
        const response = await axios.get(`${this.apiBaseUrl}/random`);
        this.selectedIdiom = response.data;
        this.showDetail = true;
      } catch (err) {
        this.error = "获取随机成语失败";
        console.error(err);
      }
    },

    copyIdiom(text) {
      navigator.clipboard.writeText(text)
          .then(() => {
            alert("成语已复制到剪贴板");
          })
          .catch(err => {
            console.error('复制失败:', err);
          });
    }
  }
};
</script>

<style>
html,
body {
  height: 100%;
  margin: 0;
  padding: 0;
}

body {
  font-family: 'Inter', system-ui, -apple-system, sans-serif;
  background-color: #f9fafb;
}

.line-clamp-2 {
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
}
</style>