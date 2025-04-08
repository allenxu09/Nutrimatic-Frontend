<template>
  <div class="bg-white rounded-xl shadow p-5 mb-6">
    <h3 class="text-lg font-semibold mb-4">正则表达式助手</h3>

    <div class="grid grid-cols-1 md:grid-cols-2 gap-4 mb-4">
      <!-- 预设模式选择 -->
      <div>
        <label class="block text-sm font-medium text-gray-700 mb-1">选择搜索模式</label>
        <select
          v-model="selectedPattern"
          class="w-full px-3 py-2 border border-gray-300 rounded-lg"
          @change="updatePattern"
        >
          <option value="">- 请选择 -</option>
          <option value="startsWith">以特定字开头</option>
          <option value="endsWith">以特定字结尾</option>
          <option value="contains">包含特定字</option>
          <option value="positionMatch">指定位置匹配</option>
          <option value="lengthMatch">指定长度成语</option>
        </select>
      </div>

      <!-- 输入区域 -->
      <div>
        <label class="block text-sm font-medium text-gray-700 mb-1">输入搜索内容</label>
        <input
          v-model="patternInput"
          type="text"
          class="w-full px-3 py-2 border border-gray-300 rounded-lg"
          :placeholder="placeholder"
          @input="updatePattern"
        />
      </div>
    </div>

    <!-- 位置选择器 - 仅在位置匹配模式下显示 -->
    <div v-if="selectedPattern === 'positionMatch'" class="mb-4">
      <label class="block text-sm font-medium text-gray-700 mb-1">字符位置</label>
      <div class="flex space-x-2">
        <input
          v-model="position"
          type="number"
          min="1"
          max="4"
          class="w-20 px-3 py-2 border border-gray-300 rounded-lg"
          @input="updatePattern"
        />
        <span class="self-center text-gray-600">（输入1-4之间的数字）</span>
      </div>
    </div>

    <!-- 生成的正则表达式 -->
    <div class="mb-4">
      <label class="block text-sm font-medium text-gray-700 mb-1">生成的正则表达式</label>
      <div class="flex">
        <input
          type="text"
          readonly
          class="w-full px-3 py-2 bg-gray-50 border border-gray-300 rounded-lg"
          :value="generatedPattern"
        />
        <button
          @click="applyPattern"
          class="ml-2 bg-blue-600 hover:bg-blue-700 text-white px-4 py-2 rounded-lg"
        >
          应用
        </button>
      </div>
    </div>

    <!-- 使用说明 -->
    <div class="text-sm text-gray-600 bg-blue-50 p-3 rounded-lg">
      <p class="font-medium mb-1">使用提示:</p>
      <ul class="list-disc list-inside space-y-1">
        <li>选择搜索模式后输入您要查找的内容</li>
        <li>【以特定字开头】例如输入"一"查找所有"一"字开头的成语</li>
        <li>【以特定字结尾】例如输入"天"查找所有以"天"字结尾的成语</li>
        <li>【包含特定字】查找包含您输入的所有字符的成语</li>
        <li>【指定位置匹配】在特定位置找到特定字，如第2个字是"人"</li>
        <li>【指定长度成语】限定成语长度，大多数成语为四字</li>
      </ul>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      selectedPattern: '',
      patternInput: '',
      position: 1,
      generatedPattern: '',
      placeholder: '请先选择搜索模式'
    };
  },
  methods: {
    updatePattern() {
      const input = this.patternInput.trim();

      switch (this.selectedPattern) {
        case 'startsWith':
          this.placeholder = '输入首字';
          this.generatedPattern = input ? `^${input}` : '';
          break;
        case 'endsWith':
          this.placeholder = '输入尾字';
          this.generatedPattern = input ? `${input}$` : '';
          break;
        case 'contains':
          this.placeholder = '输入要包含的字';
          this.generatedPattern = input;
          break;
        case 'positionMatch':
          this.placeholder = '输入特定位置的字';
          if (input && this.position >= 1 && this.position <= 4) {
            // 创建形如 .{n-1}字 的正则表达式
            this.generatedPattern = `.{${this.position - 1}}${input}`;
          } else {
            this.generatedPattern = '';
          }
          break;
        case 'lengthMatch':
          this.placeholder = '输入成语长度(通常为4)';
          this.generatedPattern = input ? `^.{${input}}$` : '';
          break;
        default:
          this.placeholder = '请先选择搜索模式';
          this.generatedPattern = '';
      }
    },
    applyPattern() {
      if (!this.generatedPattern) return;

      // 触发事件，将生成的正则表达式传递给父组件
      this.$emit('apply-pattern', this.generatedPattern);
    }
  }
};
</script>