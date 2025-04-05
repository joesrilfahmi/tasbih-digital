<template>
  <div
    class="min-h-screen bg-black flex flex-col items-center justify-center text-white p-4"
  >
    <div class="flex-1"></div>

    <!-- Counter display -->
    <div class="relative mb-8 w-full max-w-md">
      <div class="flex justify-center space-x-4 mb-6">
        <action-button @click="confirmReset" label="Reset" />
        <action-button @click="editCounter" label="Edit" />

        <div class="relative">
          <action-button
            @click="toggleDropdown"
            :label="`+${incrementValue}`"
            :has-icon="true"
          />
          <!-- Dropdown yang diperbaiki -->
          <div
            v-if="showDropdown"
            class="absolute top-full right-0 mt-1 bg-neutral-800 rounded-md shadow-lg z-10 p-2"
          >
            <div class="grid grid-cols-1 gap-2">
              <button
                v-for="value in incrementOptions"
                :key="value"
                @click="setIncrementValue(value)"
                class="px-2 py-1 rounded text-center transition-colors"
                :class="
                  incrementValue === value
                    ? 'bg-green-700 text-white'
                    : 'hover:bg-neutral-700'
                "
              >
                {{ value }}
              </button>
            </div>
          </div>
        </div>
      </div>

      <!-- Animated counter with individual digit transitions -->
      <div class="text-9xl font-light flex justify-center overflow-hidden h-32">
        <div class="flex">
          <template v-for="(digit, index) in digits" :key="index">
            <div class="relative overflow-hidden w-16 flex justify-center">
              <transition name="count-animation" mode="out-in">
                <span
                  :key="`${digit}-${prevDigitsChanged[index]}`"
                  class="block"
                >
                  {{ digit }}
                </span>
              </transition>
            </div>
          </template>
        </div>
      </div>
    </div>

    <!-- Tasbih buttons -->
    <div class="relative mb-16">
      <counter-button
        @click="increment"
        :is-active="upButtonActive"
        size="large"
        icon="up"
        label="Increment counter"
      />

      <counter-button
        @click="decrement"
        :is-active="downButtonActive"
        size="small"
        icon="down"
        label="Decrement counter"
        class="absolute -bottom-16 left-1/2 transform -translate-x-1/2"
      />
    </div>

    <div class="flex-1"></div>

    <!-- Footer -->
    <div class="py-4 text-xs text-neutral-500">
      Made by <span class="text-green-400">joesrilfahmi</span>
    </div>

    <!-- Edit modal -->
    <modal-dialog
      v-if="showEditModal"
      @close="cancelEdit"
      @save="saveEdit"
      title="Edit Counter"
    >
      <input
        type="number"
        v-model="editValue"
        class="w-full px-3 py-2 bg-neutral-700 rounded mb-4 text-white"
        min="0"
      />
    </modal-dialog>
  </div>
</template>

<script setup>
import { ref, watch, onMounted, computed, defineComponent, h } from "vue";
import Swal from "sweetalert2";
import { ChevronUp, ChevronDown } from "lucide-vue-next";

// Constants
const STORAGE_KEYS = {
  COUNT: "tasbihCount",
  INCREMENT: "tasbihIncrement",
};

// Array nilai increment yang lengkap
const INCREMENT_OPTIONS = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 15, 30, 50, 100];
const BUTTON_ACTIVE_DURATION = 300;

// State
const count = ref(100);
const prevCount = ref(100);
const incrementValue = ref(1);
const showDropdown = ref(false);
const showEditModal = ref(false);
const editValue = ref(0);
const upButtonActive = ref(false);
const downButtonActive = ref(false);
const prevDigitsChanged = ref([]);

// Computed
const digits = computed(() => count.value.toString().split(""));
const incrementOptions = computed(() => INCREMENT_OPTIONS);

// Define components using standard Vue components
const ActionButton = defineComponent({
  name: "ActionButton",
  props: {
    label: String,
    hasIcon: Boolean,
  },
  emits: ["click"],
  setup(props, { emit }) {
    return () =>
      h(
        "button",
        {
          class:
            "px-3 py-1 bg-neutral-800 rounded-md hover:bg-neutral-700 transition-colors",
          onClick: (e) => emit("click", e),
        },
        [props.label, props.hasIcon ? h("span", { class: "ml-1" }, "▼") : null]
      );
  },
});

const CounterButton = defineComponent({
  name: "CounterButton",
  props: {
    isActive: Boolean,
    size: String,
    icon: String,
    label: String,
  },
  emits: ["click"],
  setup(props, { emit }) {
    const sizeClasses = {
      large: "w-56 h-56",
      small: "w-24 h-24",
    };

    const icons = {
      up: ChevronUp,
      down: ChevronDown,
    };

    const iconSizes = {
      large: "100",
      small: "20",
    };

    const Icon = icons[props.icon];

    return () =>
      h(
        "button",
        {
          class: [
            sizeClasses[props.size],
            "rounded-full flex items-center justify-center",
            "focus:outline-none hover:bg-green-800 transition-colors",
            "cursor-pointer border-6",
            props.isActive
              ? "bg-green-700 border-green-400"
              : "bg-green-900 border-black",
          ],
          "aria-label": props.label,
          onClick: () => emit("click"),
        },
        [h(Icon, { size: iconSizes[props.size], class: "text-green-400" })]
      );
  },
});

const ModalDialog = defineComponent({
  name: "ModalDialog",
  props: {
    title: String,
  },
  emits: ["close", "save"],
  setup(props, { emit, slots }) {
    const handleBackdropClick = (e) => {
      if (e.target === e.currentTarget) emit("close");
    };

    return () =>
      h(
        "div",
        {
          class:
            "fixed inset-0 bg-black bg-opacity-70 flex items-center justify-center p-4 z-20",
          onClick: handleBackdropClick,
        },
        [
          h("div", { class: "bg-neutral-800 p-6 rounded-lg w-full max-w-xs" }, [
            h("h3", { class: "text-lg mb-4" }, props.title),
            slots.default && slots.default(),
            h("div", { class: "flex justify-end space-x-2" }, [
              h(
                "button",
                {
                  class:
                    "px-4 py-1 bg-neutral-600 rounded hover:bg-neutral-500 transition-colors",
                  onClick: () => emit("close"),
                },
                "Cancel"
              ),
              h(
                "button",
                {
                  class:
                    "px-4 py-1 bg-green-700 rounded hover:bg-green-600 transition-colors",
                  onClick: () => emit("save"),
                },
                "Save"
              ),
            ]),
          ]),
        ]
      );
  },
});

// Watchers
// Track which digits have changed
watch(count, (newValue, oldValue) => {
  updateChangedDigits(newValue, oldValue);
  saveToStorage(STORAGE_KEYS.COUNT, newValue.toString());
});

watch(incrementValue, (newValue) => {
  saveToStorage(STORAGE_KEYS.INCREMENT, newValue.toString());
});

// Methods
function updateChangedDigits(newValue, oldValue) {
  const newDigits = newValue.toString().split("");
  const oldDigits = oldValue.toString().split("");

  // Initialize with current length
  prevDigitsChanged.value = Array(newDigits.length).fill(false);

  // Check which digits changed
  for (let i = 0; i < Math.max(newDigits.length, oldDigits.length); i++) {
    // Handle different lengths
    const newIndex = newDigits.length - 1 - i;
    const oldIndex = oldDigits.length - 1 - i;

    if (
      newIndex < 0 ||
      oldIndex < 0 ||
      newDigits[newIndex] !== oldDigits[oldIndex]
    ) {
      if (newIndex >= 0) {
        prevDigitsChanged.value[newIndex] = !prevDigitsChanged.value[newIndex];
      }
    }
  }

  prevCount.value = newValue;
}

function saveToStorage(key, value) {
  localStorage.setItem(key, value);
}

function loadFromStorage(key, defaultValue) {
  const savedValue = localStorage.getItem(key);
  return savedValue !== null ? parseInt(savedValue) : defaultValue;
}

function toggleButtonActive(buttonRef) {
  buttonRef.value = true;
  setTimeout(() => {
    buttonRef.value = false;
  }, BUTTON_ACTIVE_DURATION);
}

const increment = () => {
  count.value += incrementValue.value;
  toggleButtonActive(upButtonActive);
};

const decrement = () => {
  count.value = Math.max(0, count.value - incrementValue.value);
  toggleButtonActive(downButtonActive);
};

const confirmReset = () => {
  showSwalConfirmation(
    "Reset Counter?",
    "Are you sure you want to reset the counter to zero?",
    "Yes, reset it!",
    resetCounter
  );
};

function showSwalConfirmation(title, text, confirmText, onConfirm) {
  Swal.fire({
    title,
    text,
    icon: "warning",
    showCancelButton: true,
    confirmButtonColor: "#10B981",
    cancelButtonColor: "#4B5563",
    confirmButtonText: confirmText,
    background: "#171717",
    color: "#FFFFFF",
  }).then((result) => {
    if (result.isConfirmed) {
      onConfirm();
      Swal.fire({
        title: "Reset!",
        text: "Your counter has been reset to 0.",
        icon: "success",
        showConfirmButton: false,
        timer: 1500,
        background: "#171717",
        color: "#FFFFFF",
      });
    }
  });
}

const resetCounter = () => {
  count.value = 0;
};

const toggleDropdown = (event) => {
  event.stopPropagation();
  showDropdown.value = !showDropdown.value;
};

const setIncrementValue = (value) => {
  incrementValue.value = value;
  showDropdown.value = false;
};

const editCounter = () => {
  editValue.value = count.value;
  showEditModal.value = true;
};

const saveEdit = () => {
  const newValue = parseInt(editValue.value);
  count.value = isNaN(newValue) ? 0 : Math.max(0, newValue);
  showEditModal.value = false;
};

const cancelEdit = () => {
  showEditModal.value = false;
};

function closeDropdownOnClickOutside(event) {
  if (showDropdown.value && !event.target.closest(".relative")) {
    showDropdown.value = false;
  }
}

// Lifecycle hooks
onMounted(() => {
  count.value = loadFromStorage(STORAGE_KEYS.COUNT, 100);
  prevCount.value = count.value;
  incrementValue.value = loadFromStorage(STORAGE_KEYS.INCREMENT, 1);

  // Initialize prevDigitsChanged with false for each digit
  prevDigitsChanged.value = Array(count.value.toString().length).fill(false);

  document.addEventListener("click", closeDropdownOnClickOutside);
});
</script>

<style scoped>
.count-animation-enter-active,
.count-animation-leave-active {
  transition: all 0.3s ease;
}

.count-animation-enter-from {
  opacity: 0;
  transform: translateY(30px);
}

.count-animation-leave-to {
  opacity: 0;
  transform: translateY(-30px);
}
</style>
