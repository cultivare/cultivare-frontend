<template>
  <v-combobox
    v-model="emitSelectedTags"
    v-model:search="search"
    :label="comboboxLabel"
    :items="suggestions"
    :loading="isLoading"
    multiple
    hide-no-data
    return-object
  >
    <template #selection="{ item, index }">
      <v-chip
        closable
        @click:close="removeTag(item)"
      >
        {{ item.title }}
      </v-chip>
    </template>
    <template #item="{ props, item }">
      <v-list-item v-bind="props">
        <v-list-item-title v-text="item.raw.name"></v-list-item-title>
      </v-list-item>
    </template>
  </v-combobox>
</template>

<script setup>
import { ref, watch, computed, watchEffect } from "vue";
import api from "@/services/api";
import { debounce } from "lodash";

const props = defineProps({
  modelValue: {
    type: Array,
    default: () => [],
  },
  comboboxLabel: {
    type: String,
    default: "Tags",
  },
});

const emit = defineEmits(["update:modelValue"]);

const selectedTags = ref([...props.modelValue]);
const search = ref(null);
const suggestions = ref([]);
const isLoading = ref(false);

const emitSelectedTags = computed({
  get: () => selectedTags.value,
  set: (value) => {
    selectedTags.value = value;
    emit("update:modelValue", value);
  },
});

watch(
  () => props.modelValue,
  (newVal) => {
    selectedTags.value = [...newVal];
  },
  { deep: true }
);

const onSearchInputChange = debounce(async (newSearch) => {
  console.log("[TagsCombobox] onSearchInputChange: Start", newSearch);

  if (!newSearch || newSearch.length < 2) {
    console.log(
      "[TagsCombobox] onSearchInputChange: Search input is empty or too short. Clearing suggestions."
    );
    suggestions.value = [];
    return;
  }

  isLoading.value = true;
  try {
    console.log(
      "[TagsCombobox] onSearchInputChange: Making API call with q =",
      newSearch
    );
    const response = await api.get(`/tags/autocomplete/`, {
      params: { q: newSearch },
    });
    console.log(
      "[TagsCombobox] onSearchInputChange: API response:",
      response.data
    );

    suggestions.value = response.data.filter(
      (suggestion) => !selectedTags.value.includes(suggestion)
    );

    console.log(
      "[TagsCombobox] onSearchInputChange: Updated suggestions:",
      suggestions.value
    );
  } catch (error) {
    console.error(
      "[TagsCombobox] onSearchInputChange: Error fetching suggestions:",
      error
    );
  } finally {
    console.log("[TagsCombobox] onSearchInputChange: isLoading set to false");
    isLoading.value = false;
  }
}, 350);

watchEffect(() => {
  if (search.value) {
    onSearchInputChange(search.value);
  } else {
    suggestions.value = [];
  }
});

const removeTag = (tag) => {
  console.log("[TagsCombobox] removeTag: Removing tag:", tag);
  emitSelectedTags.value = selectedTags.value.filter((t) => t !== tag);
};
</script>