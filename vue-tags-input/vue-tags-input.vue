<!--
  Entry Point for the component.
  The file contains the template and includes the script and style files.
-->

<template>
  <div
    :class="[{ 'ti-disabled': disabled }, { 'ti-focus': focused }, $attrs.class]"
    :style="$attrs.style"
    class="max-w-full"
  >
    <div class="flex flex-wrap border border-gray-200 bg-white p-1.5" :class="[autocompleteOpen ? 'rounded-t-md':'rounded-md']">
      <ul v-if="tagsCopy" class="ti-tags">
        <li
          v-for="(tag, index) in tagsCopy"
          :key="index"
          :style="tag.style"
          :class="[
            { 'ti-editing': tagsEditStatus[index] },
            tag.tiClasses,
            tag.classes,
            { 'ti-deletion-mark': isMarked(index) },
            tagColor
          ]"
          tabindex="0"
          class="rounded-md flex py-1.5 px-3 text-white ml-1.5 my-1.5"
          @click="$emit('tag-clicked', { tag, index })"
        >
          <div class="flex items-center">
            <div
              v-if="$slots['tag-left']"
              class="ti-tag-left"
            >
              <slot
                name="tag-left"
                :tag="tag"
                :index="index"
                :edit="tagsEditStatus[index]"
                :perform-save-edit="performSaveTag"
                :perform-delete="performDeleteTag"
                :perform-cancel-edit="cancelEdit"
                :perform-open-edit="performEditTag"
                :deletion-mark="isMarked(index)"
              />
            </div>
            <div :ref="setTagCenter" class="ti-tag-center">
              <span
                v-if="!$slots['tag-center']"
                :class="{ 'ti-hidden': tagsEditStatus[index] }"
                @click="performEditTag(index)"
              >{{ tag.text }}</span>
              <tag-input
                v-if="!$slots['tag-center']"
                :scope="{
                  edit: tagsEditStatus[index],
                  maxlength,
                  tag,
                  index,
                  validateTag: createChangedTag,
                  performCancelEdit: cancelEdit,
                  performSaveEdit: performSaveTag,
                }"
              />
              <slot
                name="tag-center"
                :tag="tag"
                :index="index"
                :maxlength="maxlength"
                :edit="tagsEditStatus[index]"
                :perform-save-edit="performSaveTag"
                :perform-delete="performDeleteTag"
                :perform-cancel-edit="cancelEdit"
                :validate-tag="createChangedTag"
                :perform-open-edit="performEditTag"
                :deletion-mark="isMarked(index)"
              />
            </div>
            <div
              v-if="$slots['tag-right']"
              class="ti-tag-right"
            >
              <slot
                name="tag-right"
                :tag="tag"
                :index="index"
                :edit="tagsEditStatus[index]"
                :perform-save-edit="performSaveTag"
                :perform-delete="performDeleteTag"
                :perform-cancel-edit="cancelEdit"
                :perform-open-edit="performEditTag"
                :deletion-mark="isMarked(index)"
              />
            </div>
          </div>
          <div class="ti-actions">
            <!-- dont use v-if and v-else here -> different event calling on click?! -->
            <i
              v-if="!$slots['tag-actions']"
              v-show="tagsEditStatus[index]"
              class="ti-icon-undo"
              @click="cancelEdit(index)"
            />
            <span
              v-if="!$slots['tag-actions']"
              v-show="!tagsEditStatus[index]"
              class="cursor-pointer ml-1.5"
              @click="performDeleteTag(index)"
            >&#10005;</span>
            <slot
              v-if="$slots['tag-actions']"
              name="tag-actions"
              :tag="tag"
              :index="index"
              :edit="tagsEditStatus[index]"
              :perform-save-edit="performSaveTag"
              :perform-delete="performDeleteTag"
              :perform-cancel-edit="cancelEdit"
              :perform-open-edit="performEditTag"
              :deletion-mark="isMarked(index)"
            />
          </div>
        </li>
        <li class="ti-new-tag-input-wrapper rounded-md w-32">
          <input
            ref="newTagInput"
            v-if="disableInput"
            class="ti-new-tag-input focus:ring-0 border-0 w-32"
            v-bind="$attrs"
            :class="[createClasses(newTag, tags, validation, isDuplicate)]"
            :placeholder="placeholder"
            :value="newTag"
            :maxlength="maxlength"
            :disabled="disabled"
            type="text"
            size="1"
            @keydown="performAddTags(
              filteredAutocompleteItems[selectedItem] || newTag, $event
            )"
            @paste="addTagsFromPaste"
            @keydown.delete="invokeDelete"
            @keydown.tab="performBlur"
            @keydown.up="selectItem($event, 'before')"
            @keydown.down="selectItem($event, 'after')"
            @input="updateNewTag"
            @focus="focused = true"
            @click="addOnlyFromAutocomplete ? false : selectedItem = null"
          >
        </li>
      </ul>
    </div>
    <slot name="between-elements" />
    <div
      v-if="autocompleteOpen"
      class="border h-24 overflow-y-scroll border-gray-200 border-t-0 rounded-b-md"
      @mouseout="selectedItem = null"
    >
      <slot name="autocomplete-header" />
      <ul>
        <li
          v-for="(item, index) in filteredAutocompleteItems"
          :key="index"
          :style="item.style"
          class="ti-item rounded-b-md"
          :class="[
            item.tiClasses,
            item.classes,
            isSelected(index) ? 'hover:' + tagColor:'',
          ]"
          @mouseover="disabled ? false : selectedItem = index"
        >
          <div
            v-if="!$slots['autocomplete-item']"
            @click="performAddTags(item, undefined, 'autocomplete')"
          >
            {{ item.text }}
          </div>
          <slot
            v-else
            name="autocomplete-item"
            :item="item"
            :index="index"
            :perform-add="item => performAddTags(item, undefined, 'autocomplete')"
            :selected="isSelected(index)"
          />
        </li>
      </ul>
      <slot name="autocomplete-footer" />
    </div>
  </div>
</template>

<!-- js and scss resources → I separated it into different files, because they became huge -->
<script src="./vue-tags-input.js"></script>
<style lang="scss" src="./vue-tags-input.scss" scoped></style>
