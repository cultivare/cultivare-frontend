<template>
    <v-combobox v-model="selectedParents" v-model:search="search" :items="parentItems" item-title="name" item-value="id"
        :label="dynamicComboboxLabel" multiple chips hide-selected :no-data-text="comboboxLabel"
        @update:search="onSearchChange">
        <template v-slot:selection="{ item, index }">
            <v-chip v-if="index === 0">
                <span>{{ item.raw.name }}</span>
            </v-chip>
            <span v-if="index === 1" class="text-caption grey--text">
                (+{{ selectedParents.length - 1 }} others)
            </span>
        </template>
    </v-combobox>
</template>

<script>
import api from "@/services/api";
import { debounce } from "lodash";

export default {
    name: "CultureSearch",
    props: {
        modelValue: {
            type: Array,
            default: () => [],
        },
        comboboxLabel: {
            type: String,
            default: "Parent Cultures",
        },
    },
    emits: ["update:modelValue"],
    data() {
        return {
            selectedParents: [],
            search: null,
            parentItems: [],
            loading: false,
            isFetchingInitialData: false,
        };
    },
    computed: {
        dynamicComboboxLabel() {
            if (this.parentItems.length > 0) {
                return "Select Parent(s)";
            } else if (this.search) {
                return "No results found.";
            } else {
                return "Select parent culture...";
            }
        },
    },
    watch: {
        modelValue: {
            async handler(newValue, oldValue) {
                console.log("modelValue watcher triggered:", newValue, oldValue);
                if (
                    !this.areArraysEqual(newValue, oldValue) &&
                    !this.isFetchingInitialData
                ) {
                    await this.fetchAndSetInitialParents(newValue);
                }
            },
            immediate: true,
        },
        selectedParents: {
            handler(newVal, oldVal) {
                console.log("selectedParents watcher triggered:", newVal, oldVal);
                const newIds = newVal.map((item) => item.id);
                const oldIds = oldVal ? oldVal.map((item) => item.id) : [];

                if (!this.areArraysEqual(newIds, oldIds)) {
                    this.$emit("update:modelValue", newIds);
                }
            },
            deep: true,
        },
    },
    created() {
        this.debouncedOnSearchChange = debounce(this.onSearchChange, 300);
    },
    methods: {
        areArraysEqual(arr1, arr2) {
            if (!arr1 && !arr2) return true;
            if (!arr1 || !arr2) return false;
            if (arr1.length !== arr2.length) return false;
            return arr1.every((item, index) => item === arr2[index]);
        },
        async fetchAndSetInitialParents(parentIds) {
            console.log("Fetching initial parents for IDs:", parentIds);
            if (!parentIds || parentIds.length === 0) {
                this.selectedParents = [];
                this.parentItems = [];
                return;
            }

            this.isFetchingInitialData = true;
            this.loading = true;

            try {
                // Use the /cultures/search endpoint with parent_ids
                const response = await api.get(`/cultures/search`, {
                    params: { parent_ids: parentIds },
                    paramsSerializer: {
                        indexes: null, // Use indexes: null for repeated parameters
                    },
                });
                this.selectedParents = response.data;
            } catch (error) {
                console.error("Error fetching initial data:", error);
            } finally {
                this.loading = false;
                this.isFetchingInitialData = false;
            }
        },
        onSearchChange: debounce(async function () {
            console.log("onSearchChange triggered with search term:", this.search);
            if (!this.search) {
                this.parentItems = [];
                return;
            }

            this.loading = true;
            try {
                const response = await api.get(`/cultures/search`, {
                    params: { culture_name: this.search },
                });
                this.parentItems = response.data.filter(
                    (item) => !this.selectedParents.some((sp) => sp.id === item.id)
                );
            } catch (error) {
                console.error("Error during search:", error);
                this.parentItems = [];
            } finally {
                this.loading = false;
            }
        }, 350),
    },
};
</script>