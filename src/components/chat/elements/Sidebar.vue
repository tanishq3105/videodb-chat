<template>
  <div
    :class="[
      'vdb-c-flex vdb-c-h-full vdb-c-flex-col vdb-c-gap-16 vdb-c-border-l vdb-c-bg-white vdb-c-p-16 vdb-c-pl-12 vdb-c-pr-20 vdb-c-text-black',
      {
        'vdb-c-w-1/5': !isMobile,
        'vdb-c-fixed vdb-c-left-0 vdb-c-top-0 vdb-c-z-50 vdb-c-h-full vdb-c-w-4/5 vdb-c-transform vdb-c-shadow-lg vdb-c-transition-transform vdb-c-duration-300 vdb-c-ease-in-out md:vdb-c-w-2/5':
          isMobile,
        'vdb-c--translate-x-full': isMobile && !isOpen,
      },
    ]"
  >
    <div class="vdb-c-flex vdb-c-items-center vdb-c-justify-between">
      <div class="vdb-c-text-2xl vdb-c-font-bold">
        <component :is="config.icon" />
      </div>
      <button
        v-if="isMobile"
        @click="closeSidebar"
        class="vdb-c-text-2xl vdb-c-font-bold"
      >
        &times;
      </button>
    </div>
    <Button
      variant="primary"
      class="vdb-c-px-10 vdb-c-py-12"
      :class="{
        'vdb-c-pointer-events-none vdb-c-opacity-20':
          newSessionButtonStatus !== 'active',
      }"
      @click="
        $emit('create-new-session');
        closeSidebar();
      "
    >
      <div class="vdb-c-flex vdb-c-items-center vdb-c-gap-6">
        <ComposeIcon />
        <span class="vdb-c-block vdb-c-text-sm vdb-c-font-medium"
          >New Session</span
        >
      </div>
    </Button>
    <div
      class="vdb-c-flex vdb-c-flex-grow vdb-c-flex-col vdb-c-gap-16 vdb-c-overflow-hidden"
      :class="{
        'vdb-c-pointer-events-none vdb-c-opacity-20': status === 'inactive',
      }"
    >
      <!-- Explore Agents -->
      <div
        class="vdb-c-flex vdb-c-max-h-[34%] vdb-c-flex-col vdb-c-gap-4 vdb-c-rounded-lg vdb-c-border vdb-c-border-transparent"
        :class="{ 'vdb-c-explore-agents-animation': isExploreAgentsFocused }"
      >
        <button
          @click="toggleExploreAgents()"
          :class="[
            'vdb-c-flex vdb-c-w-full vdb-c-items-center vdb-c-justify-between vdb-c-rounded-lg vdb-c-px-12 vdb-c-py-10 vdb-c-text-sm vdb-c-font-medium vdb-c-text-vdb-darkishgrey vdb-c-transition-all vdb-c-duration-300 hover:vdb-c-bg-roy',
          ]"
        >
          <div class="vdb-c-flex vdb-c-items-center vdb-c-gap-8">
            <MenuIcon class="vdb-c-mr-8" />
            <span>Explore Agents</span>
          </div>
          <ChevronDown
            :class="[
              'vdb-c-h-16 vdb-c-w-16 vdb-c-transition-transform vdb-c-duration-300',
              { 'vdb-c-rotate-180': showExploreAgents },
            ]"
            :stroke-width="1.2"
          />
        </button>
        <div
          v-if="status !== 'inactive' && showExploreAgents"
          class="vdb-c-overflow-y-scroll vdb-c-rounded-lg vdb-c-px-8 vdb-c-py-4"
          style="scrollbar-gutter: stable"
        >
          <template v-for="(agent, index) in agents" :key="index">
            <div
              @click="
                $emit('agent-click', agent);
                closeSidebar();
              "
              :class="[
                'vdb-c-cursor-pointer vdb-c-truncate vdb-c-rounded-lg vdb-c-border vdb-c-border-transparent vdb-c-bg-white vdb-c-p-8 vdb-c-text-sm vdb-c-font-medium vdb-c-text-black vdb-c-transition-all vdb-c-duration-75 hover:vdb-c-bg-[#FFF5EC]',
              ]"
            >
              <span class="vdb-c-text-vdb-orange"> @ </span>
              <span> {{ agent.name }} </span>
            </div>
          </template>
        </div>
      </div>

      <!-- Sessions -->
      <div
        class="vdb-c-flex vdb-c-max-h-[34%] vdb-c-flex-col vdb-c-overflow-hidden"
      >
        <button
          @click="toggleSessions()"
          class="vdb-c-flex vdb-c-w-full vdb-c-items-center vdb-c-justify-between vdb-c-rounded vdb-c-px-8 vdb-c-py-4 vdb-c-text-pam"
        >
          <span
            class="vdb-c-text-xs vdb-c-font-bold vdb-c-uppercase vdb-c-leading-5"
            >Sessions</span
          >
          <div class="vdb-c-p-4">
            <ChevronDown
              :class="[
                'vdb-c-h-16 vdb-c-w-16 vdb-c-transition-transform',
                { 'vdb-c-rotate-180': showSessions },
              ]"
              stroke-color="#464646"
              :stroke-width="2"
            />
          </div>
        </button>
        <div
          v-if="status !== 'inactive' && showSessions"
          class="vdb-c-mt-4 vdb-c-overflow-y-auto"
        >
          <transition name="fade" mode="out-in">
            <div v-if="addDummySession">
              <div
                class="vdb-c-cursor-pointer vdb-c-truncate vdb-c-rounded-lg vdb-c-bg-roy vdb-c-p-8 vdb-c-text-sm vdb-c-font-medium vdb-c-text-vdb-darkishgrey"
              >
                (new session)
              </div>
            </div>
          </transition>
          <transition-group name="fade" tag="div">
            <div
              v-for="session in sessions"
              :key="session.session_id"
              @click="
                $emit('session-click', session.session_id);
                closeSidebar();
              "
              @mouseenter="hoveredSession = session.session_id"
              @mouseleave="hoveredSession = null"
              :class="[
                'vdb-c-flex vdb-c-cursor-pointer vdb-c-items-center vdb-c-justify-between vdb-c-truncate vdb-c-rounded-lg vdb-c-p-8 vdb-c-px-12 vdb-c-text-sm vdb-c-font-medium vdb-c-text-vdb-darkishgrey',
                {
                  'vdb-c-bg-roy': session.session_id === selectedSession,
                  'hover:vdb-c-bg-gray-100':
                    session.session_id !== selectedSession,
                },
              ]"
            >
              <span>
                {{
                  new Date(session.created_at * 1000)
                    .toLocaleString("en-US", {
                      year: "numeric",
                      month: "2-digit",
                      day: "2-digit",
                      hour: "2-digit",
                      minute: "2-digit",
                      second: "2-digit",
                      hour12: false,
                    })
                    .replace(/\//g, ".")
                    .replace(",", " -")
                }}
              </span>
              <span
                @click.stop="
                  $emit('delete-session', session.session_id);
                  closeSidebar();
                "
                class="vdb-c-transition-all vdb-c-duration-300 hover:vdb-c-scale-110"
              >
                <DeleteIcon
                  :fill="
                    hoveredSession === session.session_id ? 'black' : '#CCCCCC'
                  "
                />
              </span>
            </div>
          </transition-group>
        </div>
      </div>

      <!-- Collections -->
      <div
        class="vdb-c-flex vdb-c-max-h-[34%] vdb-c-flex-col vdb-c-overflow-hidden"
      >
        <button
          @click="toggleCollections()"
          class="vdb-c-flex vdb-c-w-full vdb-c-items-center vdb-c-justify-between vdb-c-rounded vdb-c-px-8 vdb-c-py-4 vdb-c-text-pam"
        >
          <span
            class="vdb-c-text-xs vdb-c-font-bold vdb-c-uppercase vdb-c-leading-5"
            >Collections</span
          >
          <div class="vdb-c-p-4">
            <ChevronDown
              :class="[
                'vdb-c-h-16 vdb-c-w-16 vdb-c-transition-transform',
                { 'vdb-c-rotate-180': showCollections },
              ]"
              stroke-color="#464646"
              :stroke-width="2"
            />
          </div>
        </button>
        <div
          v-if="status !== 'inactive' && showCollections"
          class="vdb-c-mt-4 vdb-c-overflow-y-auto"
        >
          <template v-for="collection in collections" :key="collection.id">
            <div
              @click="
                $emit('collection-click', collection.id);
                closeSidebar();
              "
              :class="[
                'vdb-c-cursor-pointer vdb-c-truncate vdb-c-rounded-lg vdb-c-p-8 vdb-c-text-sm vdb-c-font-medium vdb-c-text-vdb-darkishgrey',
                {
                  'vdb-c-bg-roy':
                    showSelectedCollection &&
                    collection.id === computedSelectedCollection,
                  'hover:vdb-c-bg-gray-100':
                    collection.id !== computedSelectedCollection,
                },
              ]"
            >
              {{ collection.name }}
            </div>
          </template>
        </div>
      </div>
    </div>

    <div class="vdb-c-mt-auto vdb-c-flex vdb-c-flex-col">
      <a
        v-for="(link, index) in config.links"
        :key="index"
        class="vdb-c-mx-8 vdb-c-my-10 vdb-c-flex vdb-c-items-center vdb-c-gap-12 vdb-c-pl-8 vdb-c-text-sm vdb-c-font-medium vdb-c-text-vdb-darkishgrey hover:vdb-c-text-black hover:vdb-c-no-underline"
        :href="link.href"
        :target="link.target || '_blank'"
        rel="noopener noreferrer"
      >
        <span> {{ link.text }} </span>
        <component v-if="link.icon" :is="link.icon" />
      </a>
      <Button
        class="vdb-c-mt-16 vdb-c-w-full"
        variant="tertiary"
        :class="{
          'vdb-c-pointer-events-none vdb-c-opacity-20': status === 'inactive',
        }"
      >
        <div
          class="vdb-c-flex vdb-c-w-full vdb-c-items-center vdb-c-justify-center vdb-c-gap-6"
        >
          <a
            :href="config.primaryLink.href"
            :target="config.primaryLink.target || '_blank'"
            rel="noopener noreferrer"
            class="vdb-c-flex vdb-c-items-center vdb-c-text-white hover:vdb-c-text-white hover:vdb-c-no-underline"
          >
            <template v-if="config.primaryLink.icon">
              <img
                v-if="typeof config.primaryLink.icon === 'string'"
                :src="config.primaryLink.icon"
                alt="Primary Link Icon"
                class="vdb-c-mr-8 vdb-c-h-20 vdb-c-w-20"
              />
              <component
                v-else-if="typeof config.primaryLink.icon === 'object'"
                :is="config.primaryLink.icon"
                class="vdb-c-mr-8 vdb-c-h-20 vdb-c-w-20"
              />
            </template>
            {{ config.primaryLink.text }}
          </a>
        </div>
      </Button>
    </div>
  </div>
  <transition name="fade">
    <button
      v-if="isMobile && !isOpen"
      @click="toggleSidebar"
      class="vdb-c-shadow-md vdb-c-fixed vdb-c-left-16 vdb-c-top-16 vdb-c-z-50 vdb-c-rounded-full vdb-c-bg-white vdb-c-p-8"
    >
      <div
        class="vdb-c-flex vdb-c-h-24 vdb-c-w-24 vdb-c-flex-col vdb-c-items-center vdb-c-justify-center"
      >
        <div class="vdb-c-mb-2 vdb-c-h-2 vdb-c-w-20 vdb-c-bg-gray-600"></div>
        <div class="vdb-c-mb-2 vdb-c-h-2 vdb-c-w-20 vdb-c-bg-gray-600"></div>
        <div class="vdb-c-mb-2 vdb-c-h-2 vdb-c-w-20 vdb-c-bg-gray-600"></div>
      </div>
    </button>
  </transition>
</template>

<script setup>
import { nextTick, ref, watch, computed } from "vue";

import Button from "../../buttons/Button.vue";

import DeleteIcon from "../../icons/Delete.vue";
import ComposeIcon from "../../icons/Compose.vue";
import ChevronDown from "../../icons/ChevronDown.vue";
import MenuIcon from "../../icons/Menu.vue";

const props = defineProps({
  sessions: {
    type: Array,
    required: true,
  },
  collections: {
    type: Array,
    required: true,
  },
  agents: {
    type: Array,
    required: true,
  },
  status: {
    type: String,
    default: "active",
  },
  newSessionButtonStatus: {
    type: String,
    default: "active",
  },
  config: {
    type: Object,
    required: true,
  },
  selectedCollection: {
    type: String,
    default: "default",
  },
  selectedSession: {
    type: String,
    default: "",
  },
  addDummySession: {
    type: Boolean,
    default: false,
  },
  showSelectedCollection: {
    type: Boolean,
    default: false,
  },
  initialSessionsOpen: {
    type: Boolean,
    default: true,
  },
  initialCollectionsOpen: {
    type: Boolean,
    default: false,
  },
});

const showExploreAgents = ref(true);
const showSessions = ref(true);
const showCollections = ref(true);
const isExploreAgentsFocused = ref(false);
const exploreAgentsTimeout = ref(null);
const userClickedSessions = ref(false);
const userClickedCollections = ref(false);
const hoveredSession = ref(null);
const isMobile = ref(window.innerWidth < 1024);
const isOpen = ref(false);

const emit = defineEmits([
  "create-new-session",
  "session-click",
  "delete-session",
  "collection-click",
  "agent-click",
]);

const closeSidebar = () => {
  if (isMobile.value) {
    isOpen.value = false;
  }
};

const toggleExploreAgents = (value) => {
  showExploreAgents.value =
    value !== undefined ? value : !showExploreAgents.value;
};

const toggleSessions = (value) => {
  userClickedSessions.value = true;
  showSessions.value = value !== undefined ? value : !showSessions.value;
};

const toggleCollections = (value) => {
  userClickedCollections.value = true;
  showCollections.value = value !== undefined ? value : !showCollections.value;
};

const toggleSidebar = () => {
  isOpen.value = !isOpen.value;
};

const triggerExploreAgentsFocusAnimation = () => {
  if (exploreAgentsTimeout.value) {
    clearTimeout(exploreAgentsTimeout.value);
  }
  isExploreAgentsFocused.value = false;
  nextTick(() => {
    isExploreAgentsFocused.value = true;
    exploreAgentsTimeout.value = setTimeout(() => {
      isExploreAgentsFocused.value = false;
    }, 1000);
  });
};

const computedSelectedCollection = computed(() => {
  if (props.selectedCollection !== "default") {
    return props.selectedCollection;
  }
  return props.collections.length > 0 ? props.collections[0].id : null;
});

watch(
  () => props.initialSessionsOpen,
  (newValue) => {
    if (!userClickedSessions.value) {
      showSessions.value = newValue;
    }
  },
  { immediate: true },
);

watch(
  () => props.initialCollectionsOpen,
  (newValue) => {
    if (!userClickedCollections.value) {
      showCollections.value = newValue;
    }
  },
  { immediate: true },
);

watch(showExploreAgents, (newValue) => {
  if (newValue) {
    triggerExploreAgentsFocusAnimation();
  }
});

window.addEventListener("resize", () => {
  isMobile.value = window.innerWidth < 1024;
});

defineExpose({
  toggleExploreAgents,
  toggleSessions,
  toggleCollections,
  triggerExploreAgentsFocusAnimation,
  toggleSidebar,
});
</script>

<style>
.vdb-c-explore-agents-animation {
  animation: exploreAgentsFade 1s ease-out;
}

.vdb-c-explore-agents-animation button {
  animation: exploreAgentButtonFade 1s ease-out;
}

@keyframes exploreAgentsFade {
  0% {
    border-color: #ff7e32;
    background-color: #fff5ec;
  }
  100% {
    border-color: transparent;
    background-color: transparent;
  }
}

@keyframes exploreAgentButtonFade {
  0% {
    background-color: #ffe9d3;
  }
  100% {
    background-color: transparent;
  }
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.5s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>
