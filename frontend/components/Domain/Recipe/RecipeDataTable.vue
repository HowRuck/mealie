<template>
  <v-data-table
    v-model="selected"
    item-key="id"
    show-select
    sort-by="dateAdded"
    sort-desc
    :headers="headers"
    :items="recipes"
    :items-per-page="15"
    class="elevation-0"
    :loading="loading"
    @input="setValue(selected)"
  >
    <template #body.preappend>
      <tr>
        <td></td>
        <td>Hello</td>
        <td colspan="4"></td>
      </tr>
    </template>
    <template #item.name="{ item }">
      <a :href="`/g/${groupSlug}/r/${item.slug}`" style="color: inherit; text-decoration: inherit; " @click="$emit('click')">{{ item.name }}</a>
    </template>
    <template #item.tags="{ item }">
      <RecipeChip small :items="item.tags" :is-category="false" url-prefix="tags" />
    </template>
    <template #item.recipeCategory="{ item }">
      <RecipeChip small :items="item.recipeCategory" />
    </template>
    <template #item.tools="{ item }">
      <RecipeChip small :items="item.tools" url-prefix="tools" />
    </template>
    <template #item.userId="{ item }">
      <v-list-item class="justify-start">
        <UserAvatar :user-id="item.userId" :tooltip="false" size="40" />
        <v-list-item-content class="pl-2">
          <v-list-item-title class="text-left">
            {{ getMember(item.userId) }}
          </v-list-item-title>
        </v-list-item-content>
      </v-list-item>
    </template>
    <template #item.dateAdded="{ item }">
      {{ formatDate(item.dateAdded) }}
    </template>
  </v-data-table>
</template>

<script lang="ts">
import { computed, defineComponent, onMounted, ref, useContext } from "@nuxtjs/composition-api";
import UserAvatar from "../User/UserAvatar.vue";
import RecipeChip from "./RecipeChips.vue";
import { Recipe } from "~/lib/api/types/recipe";
import { useUserApi } from "~/composables/api";
import { UserSummary } from "~/lib/api/types/user";

const INPUT_EVENT = "input";

interface ShowHeaders {
  id: boolean;
  owner: boolean;
  tags: boolean;
  categories: boolean;
  tools: boolean;
  recipeYield: boolean;
  dateAdded: boolean;
}

export default defineComponent({
  components: { RecipeChip, UserAvatar },
  props: {
    value: {
      type: Array,
      required: false,
      default: () => [],
    },
    loading: {
      type: Boolean,
      required: false,
      default: false,
    },
    recipes: {
      type: Array as () => Recipe[],
      default: () => [],
    },
    showHeaders: {
      type: Object as () => ShowHeaders,
      required: false,
      default: () => {
        return {
          id: true,
          owner: false,
          tags: true,
          categories: true,
          recipeYield: true,
          dateAdded: true,
        };
      },
    },
  },
  setup(props, context) {
    const { $auth, i18n } = useContext();
    const groupSlug = $auth.user?.groupSlug;

    function setValue(value: Recipe[]) {
      context.emit(INPUT_EVENT, value);
    }

    const headers = computed(() => {
      const hdrs = [];

      if (props.showHeaders.id) {
        hdrs.push({ text: i18n.t("general.id"), value: "id" });
      }
      if (props.showHeaders.owner) {
        hdrs.push({ text: i18n.t("general.owner"), value: "userId", align: "center" });
      }
      hdrs.push({ text: i18n.t("general.name"), value: "name" });
      if (props.showHeaders.categories) {
        hdrs.push({ text: i18n.t("recipe.categories"), value: "recipeCategory" });
      }

      if (props.showHeaders.tags) {
        hdrs.push({ text: i18n.t("tag.tags"), value: "tags" });
      }
      if (props.showHeaders.tools) {
        hdrs.push({ text: i18n.t("tool.tools"), value: "tools" });
      }
      if (props.showHeaders.recipeYield) {
        hdrs.push({ text: i18n.t("recipe.yield"), value: "recipeYield" });
      }
      if (props.showHeaders.dateAdded) {
        hdrs.push({ text: i18n.t("general.date-added"), value: "dateAdded" });
      }

      return hdrs;
    });

    function formatDate(date: string) {
      try {
        return i18n.d(Date.parse(date), "medium");
      } catch {
        return "";
      }
    }

    // ============
    // Group Members
    const api = useUserApi();
    const members = ref<UserSummary[]>([]);

    async function refreshMembers() {
      const { data } = await api.groups.fetchMembers();
      if (data) {
        members.value = data.items;
      }
    }

    onMounted(() => {
      refreshMembers();
    });

    function getMember(id: string) {
      if (members.value[0]) {
        return members.value.find((m) => m.id === id)?.fullName;
      }

      return i18n.t("general.none");
    }

    return {
      groupSlug,
      setValue,
      headers,
      formatDate,
      members,
      getMember,
    };
  },

  data() {
    return {
      selected: [],
    };
  },
  watch: {
    value(val) {
      this.selected = val;
    },
  },
});
</script>
