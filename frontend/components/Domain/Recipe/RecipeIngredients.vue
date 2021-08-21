<template>
  <div v-if="edit || (value && value.length > 0)">
    <h2 class="mb-4">{{ $t("recipe.ingredients") }}</h2>
    <div v-if="edit">
      <draggable :value="value" handle=".handle" @input="updateIndex" @start="drag = true" @end="drag = false">
        <transition-group type="transition" :name="!drag ? 'flip-list' : null">
          <div v-for="(ingredient, index) in value" :key="generateKey('ingredient', index)">
            <v-row align="center">
              <v-text-field
                v-if="edit && showTitleEditor[index]"
                v-model="value[index].title"
                class="mx-3 mt-3"
                dense
                :label="$t('recipe.section-title')"
              >
              </v-text-field>

              <v-textarea
                v-model="value[index].note"
                class="mr-2"
                :label="$t('recipe.ingredient')"
                auto-grow
                solo
                dense
                rows="1"
              >
                <template slot="append">
                  <v-tooltip right nudge-right="10">
                    <template #activator="{ on, attrs }">
                      <v-btn icon small class="mt-n1" v-bind="attrs" v-on="on" @click="toggleShowTitle(index)">
                        <v-icon>{{ showTitleEditor[index] ? $globals.icons.minus : $globals.icons.createAlt }}</v-icon>
                      </v-btn>
                    </template>
                    <span>{{
                      showTitleEditor[index] ? $t("recipe.remove-section") : $t("recipe.insert-section")
                    }}</span>
                  </v-tooltip>
                </template>
                <template slot="append-outer">
                  <v-icon class="handle">{{ $globals.icons.arrowUpDown }}</v-icon>
                </template>
                <v-icon slot="prepend" class="mr-n1" color="error" @click="removeByIndex(value, index)">
                  {{ $globals.icons.delete }}
                </v-icon>
              </v-textarea>
            </v-row>
          </div>
        </transition-group>
      </draggable>

      <div class="d-flex row justify-end">
        <RecipeDialogBulkAdd class="mr-2" @bulk-data="addIngredient" />
        <v-btn color="secondary" dark class="mr-4" @click="addIngredient">
          <v-icon>{{ $globals.icons.create }}</v-icon>
        </v-btn>
      </div>
    </div>
    <div v-else>
      <div v-for="(ingredient, index) in value" :key="generateKey('ingredient', index)">
        <h3 v-if="showTitleEditor[index]" class="mt-2">{{ ingredient.title }}</h3>
        <v-divider v-if="showTitleEditor[index]"></v-divider>
        <v-list-item dense @click="toggleChecked(index)">
          <v-checkbox hide-details :value="checked[index]" class="pt-0 my-auto py-auto" color="secondary"> </v-checkbox>
          <v-list-item-content>
            <VueMarkdown class="ma-0 pa-0 text-subtitle-1 dense-markdown" :source="ingredient.note"> </VueMarkdown>
          </v-list-item-content>
        </v-list-item>
      </div>
    </div>
  </div>
</template>

<script>
import VueMarkdown from "@adapttive/vue-markdown";
import draggable from "vuedraggable";
import { utils } from "@/utils";
import RecipeDialogBulkAdd from "./RecipeDialogBulkAdd";
export default {
  components: {
    RecipeDialogBulkAdd,
    draggable,
    VueMarkdown,
  },
  props: {
    value: {
      type: Array,
      default: () => [],
    },

    edit: {
      type: Boolean,
      default: true,
    },
  },
  data() {
    return {
      drag: false,
      checked: [],
      showTitleEditor: [],
    };
  },
  watch: {
    value: {
      handler() {
        this.showTitleEditor = this.value.map((x) => this.validateTitle(x.title));
      },
    },
  },
  mounted() {
    this.checked = this.value.map(() => false);
    this.showTitleEditor = this.value.map((x) => this.validateTitle(x.title));
  },
  methods: {
    addIngredient(ingredients = null) {
      if (ingredients.length) {
        const newIngredients = ingredients.map((x) => {
          return {
            title: null,
            note: x,
            unit: null,
            food: null,
            disableAmount: true,
            quantity: 1,
          };
        });
        this.value.push(...newIngredients);
      } else {
        this.value.push({
          title: null,
          note: "",
          unit: null,
          food: null,
          disableAmount: true,
          quantity: 1,
        });
      }
    },
    generateKey(item, index) {
      return utils.generateUniqueKey(item, index);
    },
    updateIndex(data) {
      this.$emit("input", data);
    },
    toggleChecked(index) {
      this.$set(this.checked, index, !this.checked[index]);
    },
    removeByIndex(list, index) {
      list.splice(index, 1);
    },
    validateTitle(title) {
      return !(title === null || title === "");
    },
    toggleShowTitle(index) {
      const newVal = !this.showTitleEditor[index];
      if (!newVal) {
        this.value[index].title = "";
      }
      this.$set(this.showTitleEditor, index, newVal);
    },
  },
};
</script>

<style>
.dense-markdown p {
  margin: auto !important;
}
</style>