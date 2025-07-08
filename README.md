<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>AI聊天系统</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://cdn.jsdelivr.net/npm/font-awesome@4.7.0/css/font-awesome.min.css" rel="stylesheet">
    <script>
        tailwind.config = {
            theme: {
                extend: {
                    colors: {
                        primary: '#165DFF',
                        secondary: '#36D399',
                        neutral: '#F3F4F6',
                        'neutral-dark': '#1F2937'
                    },
                    fontFamily: {
                        sans: ['Inter', 'system-ui', 'sans-serif'],
                    },
                }
            }
        }
    </script>
    <style type="text/tailwindcss">
        @layer utilities {
            .content-auto {
                content-visibility: auto;
            }
            .scrollbar-hide::-webkit-scrollbar {
                display: none;
            }
            .scrollbar-hide {
                -ms-overflow-style: none;
                scrollbar-width: none;
            }
            .message-bubble {
                @apply rounded-lg p-4 max-w-[80%] break-words shadow-md transition-all duration-300;
            }
            .user-bubble {
                @apply bg-primary text-white self-end rounded-tr-none;
            }
            .ai-bubble {
                @apply bg-white text-neutral-dark self-start rounded-tl-none;
            }
        }
    </style>
</head>
<body class="bg-gray-50 font-sans h-screen flex flex-col">
<div id="app" class="flex-1 flex flex-col md:flex-row overflow-hidden">
    <!-- 顶部导航栏 -->
    <header class="bg-white shadow-sm px-4 py-3 flex items-center justify-between md:hidden">
        <div class="flex items-center space-x-2">
            <i class="fa fa-comment-o text-primary text-xl"></i>
            <h1 class="text-lg font-semibold text-primary">AI聊天助手</h1>
        </div>
        <button id="sidebar-toggle" class="text-gray-500">
            <i class="fa fa-bars text-xl"></i>
        </button>
    </header>

    <!-- 左侧边栏 -->
    <aside id="sidebar" class="bg-white shadow-md w-full md:w-64 h-full md:block transform -translate-x-full md:translate-x-0 transition-transform duration-300 ease-in-out fixed md:static z-20 overflow-y-auto">
        <!-- 用户信息 -->
        <div class="p-4 border-b">
            <div class="flex items-center space-x-3">
                <div class="w-10 h-10 rounded-full bg-primary flex items-center justify-center text-white">
                    <i class="fa fa-user"></i>
                </div>
                <div>
                    <h3 class="font-semibold">用户</h3>
                    <p class="text-xs text-gray-500">点击头像可更换</p>
                </div>
            </div>
        </div>

        <!-- 快捷提问 -->
        <div class="p-4">
            <h3 class="font-semibold text-gray-700 mb-3 flex items-center">
                <i class="fa fa-lightbulb-o text-primary mr-2"></i>快捷提问
            </h3>
            <div class="space-y-2">
                <div class="question-item p-2 rounded-md bg-blue-50 hover:bg-blue-100 cursor-pointer transition-colors"
                     @click="sendQuickQuestion('写一首诗歌赞美祖国')">
                    <i class="fa fa-quote-right text-blue-400 mr-2"></i>写一首诗歌赞美祖国
                </div>
                <div class="question-item p-2 rounded-md bg-blue-50 hover:bg-blue-100 cursor-pointer transition-colors"
                     @click="sendQuickQuestion('编写Java递归函数计算斐波那契数列')">
                    <i class="fa fa-code text-blue-400 mr-2"></i>编写Java递归函数
                </div>
                <div class="question-item p-2 rounded-md bg-blue-50 hover:bg-blue-100 cursor-pointer transition-colors"
                     @click="sendQuickQuestion('生成一篇300字的小学作文《我的理想》')">
                    <i class="fa fa-file-text-o text-blue-400 mr-2"></i>生成小学作文
                </div>
            </div>
        </div>

        <!-- 历史对话 -->
        <div class="p-4 border-t">
            <h3 class="font-semibold text-gray-700 mb-3 flex items-center">
                <i class="fa fa-history text-primary mr-2"></i>历史对话
            </h3>
            <div class="space-y-2">
                <div class="conversation-item p-2 rounded-md bg-gray-50 hover:bg-gray-100 cursor-pointer transition-colors flex items-center justify-between">
                    <div class="flex items-center">
                        <i class="fa fa-comment-o text-gray-400 mr-2"></i>
                        <span>新对话</span>
                    </div>
                    <div class="flex space-x-1">
                        <button class="text-xs text-gray-400 hover:text-primary" title="重命名">
                            <i class="fa fa-pencil"></i>
                        </button>
                        <button class="text-xs text-gray-400 hover:text-red-500" title="删除">
                            <i class="fa fa-trash-o"></i>
                        </button>
                    </div>
                </div>
                <div class="conversation-item p-2 rounded-md bg-gray-50 hover:bg-gray-100 cursor-pointer transition-colors flex items-center justify-between">
                    <div class="flex items-center">
                        <i class="fa fa-comment-o text-gray-400 mr-2"></i>
                        <span>Java编程学习</span>
                    </div>
                    <div class="flex space-x-1">
                        <button class="text-xs text-gray-400 hover:text-primary" title="重命名">
                            <i class="fa fa-pencil"></i>
                        </button>
                        <button class="text-xs text-gray-400 hover:text-red-500" title="删除">
                            <i class="fa fa-trash-o"></i>
                        </button>
                    </div>
                </div>
            </div>
        </div>

        <!-- 底部设置 -->
        <div class="p-4 mt-auto border-t">
            <button class="w-full py-2 px-4 bg-gray-100 hover:bg-gray-200 rounded-md transition-colors flex items-center justify-center">
                <i class="fa fa-cog text-gray-500 mr-2"></i>设置
            </button>
        </div>
    </aside>

    <!-- 主聊天区域 -->
    <main class="flex-1 flex flex-col overflow-hidden relative">
        <!-- 对话标题栏 -->
        <div class="bg-white shadow-sm px-4 py-3 flex items-center justify-between">
            <div class="flex items-center">
                <button class="md:hidden mr-3 text-gray-500" id="mobile-sidebar-toggle">
                    <i class="fa fa-bars"></i>
                </button>
                <h2 class="font-semibold text-gray-800">新对话</h2>
            </div>
            <div class="flex items-center space-x-3">
                <button class="text-gray-500 hover:text-primary transition-colors" title="复制对话">
                    <i class="fa fa-copy"></i>
                </button>
                <button class="text-gray-500 hover:text-primary transition-colors" title="下载对话">
                    <i class="fa fa-download"></i>
                </button>
                <button class="text-gray-500 hover:text-primary transition-colors" title="清空对话">
                    <i class="fa fa-trash-o"></i>
                </button>
            </div>
        </div>

        <!-- 消息列表 -->
        <div class="message-list flex-1 overflow-y-auto p-4 bg-gray-50 space-y-4 scrollbar-hide" id="message-container">
            <!-- 欢迎消息 -->
            <div class="flex items-start">
                <div class="w-8 h-8 rounded-full bg-primary/20 flex items-center justify-center text-primary mr-2 flex-shrink-0">
                    <i class="fa fa-robot"></i>
                </div>
                <div class="message-bubble ai-bubble">
                    <p>你好！我是你的AI助手，有什么可以帮助你的吗？</p>
                    <p class="text-xs text-gray-400 mt-1">你可以尝试以下提问：</p>
                    <ul class="text-xs text-gray-500 list-disc pl-5 mt-1 space-y-1">
                        <li>写一首关于春天的诗歌</li>
                        <li>解释递归算法的原理</li>
                        <li>如何提高Java程序的性能</li>
                    </ul>
                </div>
            </div>

            <!-- 动态消息将在这里渲染 -->
        </div>

        <!-- 输入区域 -->
        <div class="p-4 bg-white border-t">
            <div class="relative">
                    <textarea
                            v-model="inputContent"
                            placeholder="来说点什么吧... (Shift + Enter = 换行, '/'触发提示词)"
                            class="w-full p-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-primary/50 focus:border-primary resize-none transition-all"
                            rows="3"
                            @keydown.enter.exact="sendMessage"
                            @keydown.shift.enter="addLineBreak"
                    ></textarea>
                <div class="absolute right-3 bottom-3 flex items-center space-x-2">
                    <button class="text-gray-400 hover:text-gray-600 transition-colors" title="上传图片/文件">
                        <i class="fa fa-paperclip"></i>
                    </button>
                    <button class="bg-primary text-white py-2 px-4 rounded-lg hover:bg-primary/90 transition-colors" @click="sendMessage">
                        发送
                    </button>
                </div>
            </div>
            <p class="text-xs text-gray-400 mt-2 flex items-center">
                <i class="fa fa-info-circle mr-1"></i>
                <span>使用 <kbd class="px-1 py-0.5 bg-gray-200 rounded text-xs">/</kbd> 触发快捷提示词，使用 <kbd class="px-1 py-0.5 bg-gray-200 rounded text-xs">Shift + Enter</kbd> 换行</span>
            </p>
        </div>
    </main>

    <!-- 右侧功能面板 -->
    <aside class="hidden lg:block w-64 bg-white shadow-md border-l overflow-y-auto">
        <div class="p-4">
            <h3 class="font-semibold text-gray-700 mb-3 flex items-center">
                <i class="fa fa-cogs text-primary mr-2"></i>功能中心
            </h3>

            <!-- 功能列表 -->
            <div class="space-y-1">
                <div class="function-item p-3 rounded-md bg-primary/10 text-primary cursor-pointer flex items-center"
                     @click="switchFunction('aiChat')">
                    <i class="fa fa-comment-o mr-2"></i>
                    <span>AI聊天</span>
                </div>
                <div class="function-item p-3 rounded-md hover:bg-gray-50 cursor-pointer flex items-center transition-colors"
                     @click="switchFunction('aiDraw')">
                    <i class="fa fa-paint-brush mr-2"></i>
                    <span>AI绘画</span>
                </div>
                <div class="function-item p-3 rounded-md hover:bg-gray-50 cursor-pointer flex items-center transition-colors"
                     @click="switchFunction('codeAssistant')">
                    <i class="fa fa-code mr-2"></i>
                    <span>智能编程</span>
                </div>
                <div class="function-item p-3 rounded-md hover:bg-gray-50 cursor-pointer flex items-center transition-colors"
                     @click="switchFunction('essayCheck')">
                    <i class="fa fa-file-text-o mr-2"></i>
                    <span>作文批改</span>
                </div>
                <div class="function-item p-3 rounded-md hover:bg-gray-50 cursor-pointer flex items-center transition-colors"
                     @click="switchFunction('pptGenerator')">
                    <i class="fa fa-slideshare mr-2"></i>
                    <span>PPT生成</span>
                </div>
            </div>

            <!-- 智能体选择 -->
            <div class="mt-6">
                <h3 class="font-semibold text-gray-700 mb-3 flex items-center">
                    <i class="fa fa-user-o text-primary mr-2"></i>智能体
                </h3>
                <div class="space-y-1">
                    <div class="agent-item p-3 rounded-md bg-gray-50 hover:bg-gray-100 cursor-pointer flex items-center transition-colors">
                        <div class="w-6 h-6 rounded-full bg-blue-100 flex items-center justify-center text-blue-500 mr-2">
                            <i class="fa fa-robot text-xs"></i>
                        </div>
                        <span>AI助手</span>
                    </div>
                    <div class="agent-item p-3 rounded-md bg-gray-50 hover:bg-gray-100 cursor-pointer flex items-center transition-colors">
                        <div class="w-6 h-6 rounded-full bg-green-100 flex items-center justify-center text-green-500 mr-2">
                            <i class="fa fa-leaf text-xs"></i>
                        </div>
                        <span>花艺师</span>
                    </div>
                    <div class="agent-item p-3 rounded-md bg-red-100 text-red-600 cursor-pointer flex items-center">
                        <div class="w-6 h-6 rounded-full bg-red-200 flex items-center justify-center text-red-600 mr-2">
                            <i class="fa fa-user text-xs"></i>
                        </div>
                        <span>面试官</span>
                    </div>
                </div>
            </div>
        </div>
    </aside>
</div>

<script src="https://cdn.jsdelivr.net/npm/vue@3.3.4/dist/vue.global.prod.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/axios@1.5.1/dist/axios.min.js"></script>

<script>
    const app = Vue.createApp({
        data() {
            return {
                inputContent: '',
                messageList: [
                    {
                        id: 0,
                        type: 'ai',
                        content: '你好！我是你的AI助手，有什么可以帮助你的吗？\n\n你可以尝试以下提问：\n- 写一首关于春天的诗歌\n- 解释递归算法的原理\n- 如何提高Java程序的性能'
                    }
                ],
                currentFunction: 'aiChat',
                sidebarOpen: false
            }
        },
        mounted() {
            // 移动端侧边栏切换
            document.getElementById('sidebar-toggle').addEventListener('click', this.toggleSidebar);
            document.getElementById('mobile-sidebar-toggle').addEventListener('click', this.toggleSidebar);

            // 初始化滚动到底部
            this.scrollToBottom();
        },
        methods: {
            // 发送消息
            sendMessage() {
                if (!this.inputContent.trim()) return;

                // 添加用户消息到列表
                const userMessage = {
                    id: Date.now(),
                    type: 'user',
                    content: this.inputContent
                };
                this.messageList.push(userMessage);
                this.scrollToBottom();

                // 模拟AI回复延迟
                this.showTypingIndicator();

                setTimeout(() => {
                    let aiReply = this.generateAIResponse(this.inputContent);
                    const aiMessage = {
                        id: Date.now() + 1,
                        type: 'ai',
                        content: aiReply
                    };
                    this.messageList.pop(); // 移除"正在输入"状态
                    this.messageList.push(aiMessage);
                    this.scrollToBottom();
                }, 800);

                // 清空输入框
                this.inputContent = '';
            },

            // 显示"正在输入"状态
            showTypingIndicator() {
                this.messageList.push({
                    id: Date.now(),
                    type: 'ai',
                    content: '<div class="flex items-center"><div class="w-2 h-2 rounded-full bg-primary animate-bounce mr-1"></div><div class="w-2 h-2 rounded-full bg-primary animate-bounce mx-1" style="animation-delay: 0.2s"></div><div class="w-2 h-2 rounded-full bg-primary animate-bounce ml-1" style="animation-delay: 0.4s"></div></div>'
                });
                this.scrollToBottom();
            },

            // 生成AI回复（模拟）
            generateAIResponse(userInput) {
                // 简单的关键词匹配逻辑，实际应调用后端API
                if (userInput.includes('诗歌') || userInput.includes('赞美祖国')) {
                    return '《祖国颂》\n\n你是东方的晨曦，\n托起五千年的文明璀璨；\n你是黄河的波涛，\n激荡着无数儿女的心田。\n\n从古老的长城到澎湃的南海，\n从塞北的雪到江南的雨，\n每一寸土地都写满荣光，\n每一条河流都流淌着希望。\n\n我们在你的怀抱中成长，\n用青春和热血谱写新的篇章，\n愿你永远繁荣昌盛，\n屹立在世界的东方！';
                } else if (userInput.includes('Java') || userInput.includes('递归') || userInput.includes('斐波那契')) {
                    return '递归计算斐波那契数列的Java代码如下：\n\n```java\npublic class Fibonacci {\n    public static int fib(int n) {\n        if (n <= 1) {\n            return n;\n        }\n        return fib(n-1) + fib(n-2);\n    }\n\n    public static void main(String[] args) {\n        int n = 10;\n        System.out.println(\"斐波那契数列的第\" + n + \"项是：\" + fib(n));\n    }\n}\n```\n\n这段代码实现了递归计算斐波那契数列的功能。斐波那契数列的定义是：F(0)=0，F(1)=1, F(n)=F(n - 1)+F(n - 2)（n ≥ 2，n ∈ N*）。';
                } else if (userInput.includes('作文') || userInput.includes('理想')) {
                    return '《我的理想》\n\n每个人都有自己的理想，就像天空中的星星，点缀着美丽的夜空。我的理想是成为一名优秀的教师，用知识的阳光照亮孩子们的心灵。\n\n教师是一个神圣的职业，他们不仅传授知识，更重要的是引导学生树立正确的价值观。我希望自己能像我的老师一样，耐心地解答学生的每一个问题，鼓励他们勇敢追求自己的梦想。\n\n为了实现这个理想，我会努力学习，不断提升自己的知识水平。我也会多参加一些实践活动，积累教学经验。我相信，只要坚持不懈地努力，我的理想一定会实现。\n\n这就是我的理想，一个平凡而又伟大的理想。我会为了这个理想而努力奋斗，让它成为我人生道路上的一盏明灯。';
                } else if (userInput.includes('PPT') || userInput.includes('方案')) {
                    return '以下是一个产品推广方案PPT的大纲框架：\n\n幻灯片1：封面\n- 标题：[产品名称]推广方案\n- 副标题：打造行业新标杆\n- 日期、制作团队\n\n幻灯片2：目录\n- 市场分析\n- 产品介绍\n- 目标受众\n- 推广策略\n- 执行计划\n- 预算安排\n- 效果评估\n\n幻灯片3：市场分析\n- 行业现状\n- 市场规模与增长趋势\n- 竞争对手分析\n- 市场机会与挑战\n\n幻灯片4：产品介绍\n- 产品定位\n- 核心功能\n- 优势与特色\n- 技术参数\n- 产品展示（图片/视频）\n\n幻灯片5：目标受众\n- 目标用户群体画像\n- 用户需求与痛点\n- 购买决策因素\n\n幻灯片6-9：推广策略（分渠道）\n- 社交媒体推广\n- 搜索引擎优化\n- 内容营销\n- 线下活动\n\n幻灯片10：执行计划\n- 时间线\n- 关键节点\n- 责任分工\n\n幻灯片11：预算安排\n- 各项推广费用明细\n- 总预算\n\n幻灯片12：效果评估\n- 关键指标（流量、转化、销量等）\n- 评估周期\n- 优化调整机制\n\n幻灯片13：封底\n- 感谢语\n- 联系方式\n\n这个框架涵盖了产品推广的主要方面，可以根据实际情况进行调整和完善。需要我帮你细化某个部分的内容吗？';
                } else {
                    return '感谢你的提问！我已收到你的消息：\n\n' + userInput + '\n\n我会继续学习，不断提升自己的能力，更好地回答你的问题。如果你有其他需求，也可以告诉我哦！';
                }
            },

            // 快捷提问
            sendQuickQuestion(content) {
                this.inputContent = content;
                this.sendMessage();
            },

            // 添加换行
            addLineBreak(e) {
                e.preventDefault();
                this.inputContent += '\n';
            },

            // 切换功能
            switchFunction(functionType) {
                this.currentFunction = functionType;
                // 清空输入框
                this.inputContent = '';
                // 显示功能切换提示
                this.messageList.push({
                    id: Date.now(),
                    type: 'ai',
                    content: `已切换至${this.getFunctionName(functionType)}模式。你可以尝试：\n${this.getFunctionTips(functionType)}`
                });
                this.scrollToBottom();
            },

            // 获取功能名称
            getFunctionName(functionType) {
                const functionMap = {
                    'aiChat': 'AI聊天',
                    'aiDraw': 'AI绘画',
                    'codeAssistant': '智能编程',
                    'essayCheck': '作文批改',
                    'pptGenerator': 'PPT生成'
                };
                return functionMap[functionType] || '未知功能';
            },

            // 获取功能提示
            getFunctionTips(functionType) {
                const tipsMap = {
                    'aiChat': '输入任何问题，我会尽力为你解答',
                    'aiDraw': '描述你想要的图像（如"一只站在雪地上的红色小鸟"）',
                    'codeAssistant': '输入编程问题或需求（如"如何实现Java多线程"）',
                    'essayCheck': '粘贴作文内容，我会帮你批改并提出建议',
                    'pptGenerator': '输入主题（如"2023年度工作总结"），我会生成PPT大纲'
                };
                return tipsMap[functionType] || '';
            },

            // 切换侧边栏
            toggleSidebar() {
                this.sidebarOpen = !this.sidebarOpen;
                const sidebar = document.getElementById('sidebar');
                if (this.sidebarOpen) {
                    sidebar.classList.remove('-translate-x-full');
                    sidebar.classList.add('translate-x-0');
                } else {
                    sidebar.classList.remove('translate-x-0');
                    sidebar.classList.add('-translate-x-full');
                }
            },

            // 滚动到底部
            scrollToBottom() {
                const container = document.getElementById('message-container');
                container.scrollTop = container.scrollHeight;
            }
        }
    });

    app.mount('#app');
</script>
</body>
</html>
