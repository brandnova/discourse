<template>
  <div class="flex h-screen">
    <!-- Sidebar -->
    <div
      :class="[
        sidebarOpen ? 'w-80 absolute md:relative h-full' : 'w-0 absolute', 
        isDarkMode ? 'bg-gray-800 border-gray-700 ' : 'bg-white border-gray-200',
        'border-r overflow-y-auto hide-scrollbar transition-all duration-300'
      ]"
    >
      <div class="p-4 ps-8 flex justify-between space-x-2">
        <button @click="toggleSidebar">
          <i :class="[sidebarOpen ? 'fas fa-times' : 'fas fa-bars', isDarkMode ? 'text-white' : 'text-black', 'transition-all duration-300']"></i>
        </button>
        <div class="relative">
          <input
            type="text"
            placeholder="Search discussions"
            class="w-full pl-10 pr-4 py-2 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500 transition-all duration-300"
            :class="[isDarkMode ? 'text-gray-500 bg-gray-800 border-gray-500 focus:bg-gray-900' : 'text-black focus:bg-gray-50']"
            v-model="searchTerm"
          />
          <i class="fas fa-search absolute left-3 top-3 text-gray-400"></i>
        </div>
        
      </div>
      <div class="overflow-y-auto h-[calc(100vh-80px)]">
        <div 
          v-for="discussion in filteredDiscussions" 
          :key="discussion.id" 
          class="p-4 cursor-pointer my-2 transition-all duration-300" 
          :class="[
            isDarkMode ? 'bg-gray-700 text-white hover:bg-gray-600' : 'bg-gray-100 text-black hover:bg-gray-50',
            // currentDiscussion.id === discussion.id ? 'text-base' : '',
          ]" 
          @click="setCurrentDiscussion(discussion)"
        >
          <h3 class="font-semibold">{{ discussion.title }}</h3>
          <p class="text-sm text-gray-500 truncate">
            {{ discussion.startDate }} - {{ discussion.active ? 'Ongoing' : discussion.endDate }}
          </p>
          <p class="text-sm text-gray-500 truncate">
            {{ discussion.lastMessage }}
          </p>
        </div>

      </div>
    </div>

    <!-- Main Chat Section -->
    <div class="flex-1 w-auto flex flex-col transition-all duration-300"
      :class="[
        isDarkMode ? 'bg-gray-600 text-white' : ' bg-gray-100 text-black',
        // currentDiscussion.id === discussion.id ? 'text-base' : '',
      ]"
    >
      <!-- Chat Header -->
      <div class="border-b p-4 px-8 flex items-center justify-between transition-all duration-300"
        :class="[
          isDarkMode ? 'bg-gray-800 border-gray-700 text-white' : 'bg-white border-gray-200 text-black',
          // currentDiscussion.id === discussion.id ? 'text-base' : '',
        ]" 
      >
        <div class="flex items-center space-x-5">
          <button @click="toggleSidebar" class="mr-4 focus:outline-none">
            <i :class="[sidebarOpen ? 'fas fa-times md:hidden' : 'fas fa-bars']"></i>
          </button>
          <div>
            <h2 class="font-semibold">{{ currentDiscussion.title }}</h2>
            <p class="text-sm text-gray-500">
              {{ currentDiscussion.active ? 'Active' : 'Inactive' }}
            </p>
          </div>
        </div>
        <div class="flex items-center">
          <button @click="toggleDarkMode" class="p-2 rounded-full transition-all duration-300" 
            :class="[isDarkMode ? 'hover:bg-gray-700' : 'hover:bg-gray-300']"
          >
            <i class="transition-all duration-300" :class="[isDarkMode ? 'fas fa-sun text-yellow-400' : 'fas fa-moon text-gray-600']"></i>
          </button>
        </div>
      </div>

      <!-- Chat Messages -->
      <div class="flex-1 overflow-y-auto p-5 hide-scrollbar" ref="messageContainer">
        <div v-for="message in currentDiscussion.messages" :key="message.id" class="mb-4" :class="{ 'text-right': message.sender === 'You' }">
          <div class="shadow-md max-w-96 min-w-28 break-words whitespace-normal inline-block p-3 rounded-lg text-left transition-all duration-300"
            :class="[
              // Handling the case when the sender is 'You'
              message.sender === 'You' 
                ? (isDarkMode 
                    ? 'bg-gray-800 text-white'  // Sender is 'You' and dark mode is active
                    : 'bg-blue-500 text-white') // Sender is 'You' and dark mode is inactive

                // Handling the case when the sender is not 'You'
                : (isDarkMode 
                    ? 'bg-gray-800 text-white'  // Sender is not 'You' and dark mode is active
                    : 'bg-white text-black'),   // Sender is not 'You' and dark mode is inactive
            ]">
            <p class="font-semibold mb-1">{{ message.sender }}</p>
            <p v-html="formatMessage(message.content)"
            class="p-2 rounded-lg transition-all duration-300"
              :class="[
              message.sender === 'You' 
                ? (isDarkMode 
                    ? 'bg-gray-500 bg-opacity-25'  // Sender is 'You' and dark mode is active
                    : 'bg-blue-700 bg-opacity-25') // Sender is 'You' and dark mode is inactive

                // Handling the case when the sender is not 'You'
                : (isDarkMode 
                    ? 'bg-gray-500 bg-opacity-25'  // Sender is not 'You' and dark mode is active
                    : 'bg-gray-300 bg-opacity-25'), // Sender is not 'You' and dark mode is inactive
              ]"></p>
            <p class="text-xs mt-1 transition-all duration-300" 
              :class="[
                isDarkMode ? 'text-gray-500' : 'text-gray-800', 
                message.sender === 'You' ? 'text-left' : 'text-right'
              ]">
              {{ formatTime(message.timestamp) }}
            </p>
          </div>
        </div>
        <button
          v-if="showScrollToBottom"
          @click="scrollToBottom"
          class="fixed right-10 bottom-40 p-3 bg-opacity-75 motion-safe:animate-bounce text-black rounded-full shadow-md"
            :class="[isDarkMode ? 'bg-gray-500' : 'bg-gray-300']"
          >
          <i class="fas fa-chevron-down"
            :class="[isDarkMode ? 'text-white' : 'text-black']"
          ></i>
        </button>
      </div>

      <!-- Chat Input -->
      <div class="border-t p-4 px-8 transition-all duration-300"
      :class="[
          isDarkMode ? 'bg-gray-800 border-gray-800' : 'bg-white border-gray-200',
          // currentDiscussion.id === discussion.id ? 'text-base' : '',
        ]"
      >
        <div v-if="currentDiscussion.active">
          <div class="flex mb-2 justify-between">
            <span>
              <button
                v-for="(tool, index) in editingTools"
                :key="index"
                @click="applyFormatting(tool.tag)"
                class="mr-2 p-1 rounded hover:bg-gray-200 focus:outline-none"
                :title="tool.name"
              >
                <i :class="['fas', tool.icon, 'text-gray-600']"></i>
              </button>
            </span>
            <p v-if="typingUser" class="mr-4 text-sm text-gray-500">
              {{ typingUser }} is typing...
            </p>
            
          </div>
          <div class="flex items-center">
            <textarea
              ref="messageInput"
              v-model="newMessage"
              @input="handleInput"
              @keydown.enter.prevent="sendMessage"
              placeholder="Type a message"
              class="flex-1 border border-gray-300 rounded-lg px-4 py-2 mr-2 focus:outline-none focus:ring-2 focus:ring-blue-500 dark:bg-gray-700 dark:text-white resize-none transition-all duration-300"
              :class="[isDarkMode ? 'text-gray-500 bg-gray-800 border-gray-500 focus:bg-gray-900' : 'text-black focus:bg-gray-50']"
              rows="2"
            ></textarea>
            <button
              class="p-2 w-10 bg-blue-500 text-white rounded-lg hover:bg-blue-600 focus:outline-none"
              @click="sendMessage"
            >
              <i class="fas fa-paper-plane"></i>
            </button>
          </div>
        </div>
        <div v-else class="text-center text-gray-500 dark:text-gray-400">
          This discussion has ended.
        </div>
      </div>
    </div>
    
  </div>
</template>

<script>
import { ref, computed, onMounted, onUnmounted, nextTick } from 'vue';
import { format } from 'date-fns';
// import axios from 'axios';  // Uncomment when using axios for API calls

export default {
  name: 'DiscussionRoom',
  setup() {
    const discussions = ref([
      {
        id: 1,
        title: 'General Discussion',
        lastMessage: 'What do you think about the new feature?',
        active: true,
        startDate: '2023-06-01',
        endDate: null,
        messages: [
          { id: 1, sender: 'John Doe', content: 'Hello everyone!', timestamp: new Date(2023, 5, 1, 14, 30) },
          { id: 2, sender: 'You', content: 'Hi John, welcome to the discussion!', timestamp: new Date(2023, 5, 1, 14, 32) },
        ]
      },
      {
        id: 2,
        title: 'Technical Support',
        lastMessage: 'Has anyone encountered this bug?',
        active: true,
        startDate: '2023-06-15',
        endDate: null,
        messages: []
      },
      {
        id: 3,
        title: 'Project Ideas',
        lastMessage: 'Let\'s brainstorm some new concepts',
        active: false,
        startDate: '2023-05-01',
        endDate: '2023-05-31',
        messages: []
      },
    ]);

    const currentDiscussion = ref(discussions.value[0]);
    const searchTerm = ref('');
    const sidebarOpen = ref(true);
    const newMessage = ref('');
    const isDarkMode = ref(false);
    const showScrollToBottom = ref(false);
    const messageContainer = ref(null);
    const messageInput = ref(null);
    const typingUser = ref(null);
    const typingTimeout = ref(null);

    const editingTools = [
      { name: 'Bold', icon: 'fa-bold', tag: 'b' },
      { name: 'Italic', icon: 'fa-italic', tag: 'i' },
      { name: 'Underline', icon: 'fa-underline', tag: 'u' },
    ];

    const filteredDiscussions = computed(() => {
      return discussions.value.filter(discussion =>
        discussion.title.toLowerCase().includes(searchTerm.value.toLowerCase())
      );
    });

    const setCurrentDiscussion = (discussion) => {
      currentDiscussion.value = discussion;
    };

    const toggleSidebar = () => {
      sidebarOpen.value = !sidebarOpen.value;
    };

    const toggleDarkMode = () => {
      isDarkMode.value = !isDarkMode.value;
      localStorage.setItem('darkMode', isDarkMode.value);
    };

    const formatTime = (timestamp) => {
      return format(timestamp, 'HH:mm');
    };

    const formatMessage = (content) => {
      // Basic HTML sanitization (should be more robust in production)
      return content.replace(/</g, '&lt;').replace(/>/g, '&gt;');
    };

    const applyFormatting = (tag) => {
      const textarea = messageInput.value;
      const start = textarea.selectionStart;
      const end = textarea.selectionEnd;
      const selectedText = newMessage.value.slice(start, end);
      const replacement = `<${tag}>${selectedText}</${tag}>`;
      newMessage.value = newMessage.value.slice(0, start) + replacement + newMessage.value.slice(end);
      nextTick(() => {
        textarea.focus();
        textarea.setSelectionRange(start + tag.length + 2, start + replacement.length - tag.length - 3);
      });
    };

    const sendMessage = () => {
      if (newMessage.value.trim() !== '') {
        const message = {
          id: currentDiscussion.value.messages.length + 1,
          sender: 'You',
          content: newMessage.value,
          timestamp: new Date(),
        };
        currentDiscussion.value.messages.push(message);
        newMessage.value = '';
        scrollToBottom();
        // sendMessageToAPI(message);  // Uncomment when API is ready
      }
    };

    const scrollToBottom = () => {
      nextTick(() => {
        if (messageContainer.value) {
          messageContainer.value.scrollTop = messageContainer.value.scrollHeight;
        }
      });
    };

    const handleScroll = () => {
      if (messageContainer.value) {
        const { scrollTop, scrollHeight, clientHeight } = messageContainer.value;
        showScrollToBottom.value = scrollTop < scrollHeight - clientHeight - 100;
      }
    };

    const handleInput = () => {
      // Simulate typing indicator
      typingUser.value = 'You';
      clearTimeout(typingTimeout.value);
      typingTimeout.value = setTimeout(() => {
        typingUser.value = null;
      }, 1500);

      // In a real app, you would send a typing event to the server here
      // sendTypingEventToAPI();
    };

    onMounted(() => {
      if (messageContainer.value) {
        messageContainer.value.addEventListener('scroll', handleScroll);
      }
      isDarkMode.value = localStorage.getItem('darkMode') === 'true';
      // fetchDiscussionsFromAPI();  // Uncomment when API is ready
    });

    onUnmounted(() => {
      if (messageContainer.value) {
        messageContainer.value.removeEventListener('scroll', handleScroll);
      }
    });

    // API integration examples (commented out)

    /*
    const fetchDiscussionsFromAPI = async () => {
      try {
        const response = await axios.get('/api/discussions');
        discussions.value = response.data;
      } catch (error) {
        console.error('Error fetching discussions:', error);
      }
    };

    const sendMessageToAPI = async (message) => {
      try {
        await axios.post(/api/discussions/${currentDiscussion.value.id}/messages, message);
      } catch (error) {
        console.error('Error sending message:', error);
      }
    };

    const sendTypingEventToAPI = async () => {
      try {
        await axios.post(/api/discussions/${currentDiscussion.value.id}/typing, { userId: 'currentUserId' });
      } catch (error) {
        console.error('Error sending typing event:', error);
      }
    };
    */

    return {
      discussions,
      currentDiscussion,
      searchTerm,
      sidebarOpen,
      newMessage,
      isDarkMode,
      showScrollToBottom,
      messageContainer,
      messageInput,
      typingUser,
      editingTools,
      filteredDiscussions,
      setCurrentDiscussion,
      toggleSidebar,
      toggleDarkMode,
      formatTime,
      formatMessage,
      applyFormatting,
      sendMessage,
      scrollToBottom,
      handleInput,
    };
  }
}
</script>