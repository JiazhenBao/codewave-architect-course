<template>
  <div
    :class="[
      $style.root,
      isPreview ? $style.preview : '',
      isPreview && !$env.VUE_APP_DESIGNER ? $style.disEvent : '',
    ]"
    :color="color || (formItemVM && formItemVM.color)"
    :readonly="readonly"
    :disabled="currentDisabled"
    :opened="popperOpened"
    :clearable="
      clearable && !!(filterable ? filterText || currentText : currentText)
    "
    :multiple="multiple"
    :multiple-tags="multiple && multipleAppearance === 'tags'"
    :prefix="prefix ? prefix : undefined"
    :suffix="suffix ? suffix : undefined"
    :start="!!prefix"
    :end="!!suffix"
    :tabindex="readonly || currentDisabled ? '' : 0"
    @click="focus"
    @keydown.up.prevent="$refs.popper.currentOpened ? shift(-1) : open()"
    @keydown.down.prevent="$refs.popper.currentOpened ? shift(+1) : open()"
    @keydown.enter.stop.prevent="onEnter"
    @keydown.esc.stop="close(), (filterText = '')"
    @keydown.delete.stop="onDelete"
    @blur="onRootBlur">
    <span :class="$style.baseline">b</span
    ><!-- 用于基线对齐 -->
    <span
      v-show="
        !isItemDisplay ||
        (!filterText &&
          (multiple ? !selectedVMs.length : !selectedVM) &&
          !compositionInputing)
      "
      :class="$style.placeholder"
      >{{ isPreview ? '--' : placeholder }}</span
    >
    <span
      v-if="prefix"
      :class="$style.prefix"
      :name="prefix"
      @click="$emit('click-prefix', $event, this)"
      >
        <slot name="prefix" >
          <i-ico :name="prefix" v-if="prefix!=='search'" />
        </slot
    ></span>
    <div
      v-show="isItemDisplay"
      :class="[$style.text, !multiple && $style.textEllipsis]"
      v-ellipsis-title
      :tags-overflow="tagsOverflow"
      :style="{ direction: ellipsisDirection }"
      ref="inputOuter">
      <!-- @override: 添加了flag功能 -->
      <slot name="flag">
        <span
          v-if="selectedVM && selectedVM.flag !== undefined"
          :class="$style.flag"
          :layer="selectedVM && selectedVM.layer"
          v-tooltip.top="selectedVM && selectedVM.flag"></span>
      </slot>
      <span :class="$style.label" v-if="label || $slots.label">
        {{ label }}
        <slot name="label"></slot>
      </span>
      <template v-if="!multiple && !filterable">
        <template v-if="selectedVM">
          <slot
            name="selected"
            :item="selectedVM.item"
            :value="selectedVM.value"
            :text="selectedVM.text">
            <f-render
              :vnode="
                selectedVM &&
                (selectedVM.$slots && selectedVM.$slots.default
                  ? selectedVM.$slots.default
                  : [$at2(selectedVM, field || textField)])
              "></f-render>
          </slot>
        </template>
      </template>
      <span v-else-if="multipleAppearance === 'text'">{{ currentText }}</span>
      <template v-else-if="multipleAppearance === 'tags'">
        <template
          v-if="tagsOverflow === 'hidden' || tagsOverflow === 'visible'">
          <span
            :class="$style.tag"
            v-for="(itemVM, index) in selectedVMs"
            :key="duplicated ? itemVM.value + '_' + index : itemVM.value"
            :ref="`item_${index}`">
            <span
              :class="[$style['tag-text'], iconField ? $style.iconwrap : '']">
              <img :class="$style.icon" v-if="iconField && itemVM.icon && typeof itemVM.icon === 'string'" :src="itemVM.icon" />
              {{ itemVM.currentText }}
            </span>
            <span
              v-show="!isPreview"
              :class="$style['tag-remove']"
              @click.stop="removeTag(itemVM, false)"></span>
          </span>
        </template>
        <template v-else-if="tagsOverflow === 'collapse'">
          <span
            :class="$style.tag"
            v-for="(itemVM, index) in selectedVMs"
            :key="duplicated ? itemVM.value + '__' + index : itemVM.value"
            :ref="`item_${index}`">
            <span
              :class="[$style['tag-text'], iconField ? $style.iconwrap : '']"
              ><img
                :class="$style.icon"
                v-if="iconField && itemVM.icon && typeof itemVM.icon === 'string'"
                :src="itemVM.icon" />{{ itemVM.currentText }}</span
            >
            <span
              v-show="!isPreview"
              :class="$style['tag-remove']"
              @click.stop="removeTag(itemVM, false)"></span>
          </span>
          <span
            :class="$style.tag"
            v-if="
              selectedVMs.length - collapseCounter >= 1 &&
              selectedVMs.length !== 1
            "
            :style="{ 'vertical-align': 'middle', 'padding-right': '6px' }">
            <span :class="$style['tag-text']"
              >+{{ selectedVMs.length - collapseCounter }}...</span
            >
          </span>
        </template>
      </template>
      <u-input
        v-if="filterable"
        :class="$style.input"
        ref="input"
        :readonly="readonly"
        :disabled="currentDisabled"
        :filterable="filterable"
        :multiple-tags="multiple && multipleAppearance === 'tags'"
        :value="filterText"
        @input="onInput"
        @focus="onFocus"
        @blur="onBlur"
        @keydown.delete.stop="onInputDelete"
        :style="{ width: multiple && inputWidth + 'px' }"
        @compositionstart="compositionInputing = true"
        @compositionend="compositionInputing = false">
      </u-input>
    </div>
    <span
      v-if="suffix"
      :name="suffix"
      :class="$style.suffix"
      @click="$emit('click-suffix', $event, this)"
      ><slot name="suffix"></slot
    >
      <slot name="suffix" >
        <i-ico :name="suffix" v-if="suffix!=='search'" />
      </slot>
    </span>
    <span
      v-if="
        !currentDisabled &&
        !readonly &&
        clearable &&
        !!(filterable ? filterText || currentText : currentText)
      "
      :class="$style.clearable"
      @click.stop="clear"></span>
    <m-popper
      :class="$style.popper"
      ref="popper"
      :color="color"
      :placement="placement"
      :append-to="appendTo"
      :disabled="readonly || currentDisabled"
      :style="{ width: currentPopperWidth }"
      :footer="showRenderFooter"
      @update:opened="$emit('update:opened', $event, this)"
      @before-open="$emit('before-open', $event, this)"
      @before-close="$emit('before-close', $event, this)"
      @open="onOpen"
      @close="onClose"
      @before-toggle="$emit('before-toggle', $event, this)"
      @toggle="$emit('toggle', $event, this)"
      @click.stop
      @scroll.stop="showRenderFooter ? null : onScroll($event)"
      @mousedown.stop>
      <div
        :class="$style.wrap"
        @scroll.stop="showRenderFooter ? onScroll($event) : null"
        ref="popperwrap">
        <u-select-item-all-check
          v-if="hasAllCheckItem && multiple"
          :all-checked="allChecked"
          :parent-v-m="this"
          :check-all="checkAll"
          >{{ allCheckItemText }}</u-select-item-all-check
        >
        <slot></slot>
        <template v-if="currentData">
          <div
            :class="$style.status"
            key="empty"
            v-if="!currentData.length && !currentLoading && showEmptyText">
            <slot name="empty">{{ emptyText }}</slot>
          </div>
          <template v-else>
            <component
              :is="ChildComponent"
              v-for="(item, index) in currentData"
              v-if="item !== undefined || item !== null"
              :key="
                filterable
                  ? $at2(item, valueField) + '_' + index
                  : $at2(item, valueField)
              "
              :text="$at2(item, field || textField)"
              :value="$at2(item, valueField)"
              :disabled="item.disabled || disabled"
              :item="item"
              :description="description ? $at2(item, descriptionField) : null"
              :icon="$at2(item, iconField)">
              <slot
                name="text"
                :item="item"
                :text="$at2(item, field || textField)"
                :value="$at2(item, valueField)"
                :disabled="item.disabled || disabled"
                :description="description ? $at2(item, descriptionField) : null"
                :icon="$at2(item, iconField)">
                <span :class="$style.iconwrap" v-if="iconField">
                  <img
                    :class="$style.icon"
                    v-if="$at2(item, iconField)"
                    :src="$at2(item, iconField)" />
                  {{ $at2(item, field || textField) }}
                </span>
                <template v-else>
                  {{ $at2(item, field || textField) }}
                </template>
              </slot>
            </component>
          </template>
        </template>
        <div :class="$style.status" status="loading" v-if="currentLoading">
          <slot name="loading"><u-spinner></u-spinner> {{ loadingText }}</slot>
        </div>
      </div>
      <div
        :class="$style.footer"
        v-if="showRenderFooter"
        vusion-slot-name="renderFooter"
        ref="footer">
        <slot name="renderFooter">
          <s-empty
            v-if="
              !$slots.renderFooter &&
              $env.VUE_APP_DESIGNER &&
              !!$attrs['vusion-node-path']
            ">
          </s-empty>
        </slot>
      </div>
    </m-popper>
  </div>
</template>

<script>
import { sync } from '@lcap/vue2-utils';
import { UListView } from '../u-list-view.vue';
import { ellipsisTitle } from '../../directives';
import i18n from './i18n';
import DataSource from '../../utils/DataSource';
import DataSourceNew from '../../utils/DataSource/new';
import MPreview from '../u-text.vue/preview';
import AllCheck from './allCheck.vue';
import i18nMixin from '../../mixins/i18n';

export default {
    name: 'u-select',
    component: {
        'u-select-item-all-check': AllCheck,
    },
    childName: 'u-select-item',
    groupName: 'u-select-group',
    isSelect: true,
    directives: { ellipsisTitle },
    extends: UListView,
    // i18n,
    mixins: [
      i18nMixin('u-select'),
      MPreview,
      sync({
        opened: 'popperOpened',
        preview: 'isPreview',
      })
    ],
    props: {
        // @inherit: value: { type: String, default: '' },
        // @inherit: value: Array,
        // @inherit: field: String,
        // @inherit: textField: { type: String, default: 'text' },
        // @inherit: valueField: { type: String, default: 'value' },
        // @inherit: data: Array,
        // @inherit: dataSource: [DataSource, Function, Object],
        // @inherit: cancelable: { type: Boolean, default: false },
        // @inherit: multiple: { type: Boolean, default: false },
        // @inherit: keepOrder: { type: Boolean, default: false },
        autofocus: { type: Boolean, default: false },
        duplicated: { type: Boolean, default: false },
        multipleAppearance: { type: String, default: 'tags' },
        tagsOverflow: { type: String, default: 'collapse' },
        autoSelect: { type: Boolean, default: false },
        emptyValueIsNull:{ type: Boolean, default: false},
        placeholder: { type: String, default: '请选择' },
        clearable: { type: Boolean, default: false },
        filterable: { type: Boolean, default: false },
        preview: { type: Boolean, default: false },
        matchMethod: { type: [String, Function], default: 'includes' },
        caseSensitive: { type: Boolean, default: false }, // @inherit: loadingText: { type: String, default: '加载中...' },
        placement: { type: String, validator: (value) => /^(top|bottom|left|right)(-start|-end)?$/.test(value) },
        emptyText: {
            type: String,
            default() {
                return this.$tt('empty');
            },
        },
        emptyDisabled: { type: Boolean, default: false }, // @inherit: initialLoad: { type: Boolean, default: true },
        // @inherit: pageable: { type: Boolean, default: false },
        // @inherit: pageSize: { type: Number, default: 50 },
        // @inherit: remotePaging: { type: Boolean, default: false },
        remoteFiltering: { type: Boolean, default: false },
        autoComplete: { type: Boolean, default: false },
        opened: { type: Boolean, default: false },
        prefix: String,
        suffix: String,
        label: { type: String, default: '' },
        ellipsisDirection: { type: String, default: 'ltr' },
        appendTo: {
            type: String,
            default: 'reference',
            validator: (value) => ['body', 'reference'].includes(value),
        },
        color: String,
        popperWidth: { type: String, default: '' },
        showEmptyText: { type: Boolean, default: true }, // 控制是否展示emptyText
        descriptionField: String,

    // 新增用来分页
    pagination: { type: Boolean, default: undefined },
    sorting: { type: Object },
    dataSource: [DataSource, DataSourceNew, Function, Object, Array],
    description: { type: Boolean, default: false },
    showRenderFooter: { type: Boolean, default: false }, // 可扩展下拉项
    iconField: String,
    hasAllCheckItem: { type: Boolean, default: false },
    allCheckItemText: { type: String, default: '全选' },

    isItemDisplay: { type: Boolean, default: true },
    autoCheckSelectedValue: { type: Boolean, default: true },
  },
  data() {
    return {
      // @inherit: ChildComponent: this.$options.childName,
      // @inherit: groupVMs: [],
      // @inherit: itemVMs: [],
      // @inherit: selectedVM: undefined,
      // @inherit: selectedVMs: undefined,
      focusedVM: undefined, // @inherit: currentMultiple: this.multiple,
      // @inherit: currentDataSource: undefined,
      // @inherit: currentLoading: false,
      currentText: '', // 显示文本
      filterText: '', // 过滤文本，只有 input 时会改变它
      filterInputFocused: false,
      preventBlur: false,
      inputWidth: 20,
      popperOpened: false,
      currentPopperWidth: this.popperWidth || '100%',
      compositionInputing: false,
      collapseCounter: 0,
      selectedDataQueue: [null, null], // 选中不在第一页数据处理
    };
  },
  computed: {
    currentDisabled() {
      if (this.disabled) return true;
      else if (this.emptyDisabled)
        return this.currentData
          ? !this.currentData.length
          : !this.itemVMs.length;
      else return false;
    },
    filtering() {
      return {
        [this.field || this.textField]: {
          operator: this.matchMethod,
          value: this.filterText,
          caseInsensitive: !this.caseSensitive,
        },
      };
    },
    paging() {
      if (this.usePagination) {
        const paging = {};
        paging.size = this.pageSize === '' ? 50 : this.pageSize;
        paging.number = paging.number || 1;
        return paging;
      } else return undefined;
    },
    usePagination() {
      if (typeof this.pagination === 'undefined') {
        return this.pageable === true;
      }

      return !!this.pagination;
    },
    allChecked() {
      // readme:copy from u-list-view/index, changed some behavior for u-select function;
      let checkedLength = 0;
      let disabledUnCheckedItemCount = 0;
      this.itemVMs.forEach((itemVM) => {
        if (itemVM.currentSelected) checkedLength++;
        else if (itemVM.disabled || itemVM.readonly)
          disabledUnCheckedItemCount++;
      });

      if (!this.hasAllCheckItem) {
        disabledUnCheckedItemCount = 0;
      }
      if (checkedLength === 0) return false;
      else if (
        checkedLength + disabledUnCheckedItemCount ===
        this.itemVMs.length
      )
        return true;
      else return null;
    },
  },
  watch: {
    filterText(filterText) {
      this.inputWidth = (filterText || '').length * 12 + 20;
    },
    opened(opened) {
      if (opened === this.popperOpened) return;
      this.toggle(opened);
    },
    appendTo(appendTo) {
      this.setPopperWidth();
    },
    value(value) {
      if (this.autoCheckSelectedValue) {
        this.setSelectedDataQueue(value);
      }

      if (
        this.filterable &&
        !this.multiple &&
        this.currentDataSource &&
        Array.isArray(this.currentDataSource.data)
      ) {
        const selectedItem = this.currentDataSource.data.find(
          (d) => this.$at2(d, this.valueField) === value,
        );

        const filterText = selectedItem
          ? this.$at2(selectedItem, this.field || this.textField)
          : '';
        if (filterText !== this.filterText) {
          this.filterText = filterText;
        }
      }
    },
    async selectedDataQueue(value) {
      if (!this.autoCheckSelectedValue) return;

      // 当value有值，并且已经加载过一次数据才能开启判断
      // value 和 data 可能都是异步的，所以需要watch
      // 只有数据源是load函数的情况才需要处理
      const hasData = (value || []).every((item) => !!item);
      if (hasData && value.length === 2) {
        this.loadMoreForSelected();
      }
    },
    popperWidth() {
      this.setPopperWidth();
    },
  },
  created() {
    this.$watch('selectedVM', (selectedVM, oldVM) => {
      if (selectedVM && oldVM && selectedVM.currentText === oldVM.currentText) {
        return;
      }
      if (this.filterable) {
        if (this.selectedVM && !this.filterInputFocused) {
          this.filterText = this.selectedVM.currentText;
        } else if (!this.value && !this.filterInputFocused) {
          // 响应this.value 的变化 = '' 时处理 清空
          this.filterText = '';
          this.currentText = '';
        }
        // blur 事件会处理这个未搜索到置空的问题
        // this.filterText = ? this.selectedVM.currentText : '';
      } else {
        this.currentText = this.selectedVM ? this.selectedVM.currentText : '';
      }
    });
    this.$watch('selectedVMs', (selectedVMs) => {
      this.currentText = selectedVMs
        .map((itemVM) => itemVM.currentText)
        .join(', ');
      const popperVM = this.$refs.popper;
      popperVM && popperVM.currentOpened && popperVM.scheduleUpdate();
      // 计算折叠时，最多能展示几个标签
      if (this.tagsOverflow === 'collapse' || this.tagsOverflow === 'hidden') {
        this.$nextTick(() => {
          this.collapseCounter = 0;
          const collapseTagWidth = this.tagsOverflow === 'collapse' ? 34 : 0;
          const marginWidth = 4;
          let lastAddElementWidth = 0;
          const inputElWidth = this.inputWidth || 0;
          // 预留出"+N"的标签宽度
          let inputWidth =
            this.$refs.inputOuter.offsetWidth - inputElWidth - collapseTagWidth;
          // 先计算前N-1个元素长度是否超出输入框
          for (let i = 0; i < this.selectedVMs.length - 1; i++) {
            if (this.$refs[`item_${i}`]) {
              this.$refs[`item_${i}`][0].style.display = 'inline-block';
              this.$refs[`item_${i}`][0].style['vertical-align'] = 'middle';
              const itemWidth =
                this.$refs[`item_${i}`][0].offsetWidth + marginWidth;
              if (inputWidth - itemWidth < 0) {
                break;
              }
              inputWidth -= itemWidth;
              this.collapseCounter += 1;
            }
          }
          // 计算最后一个元素能否加入输入框
          this.$nextTick(() => {
            const lastItem = this.$refs[`item_${this.selectedVMs.length - 1}`];
            if (lastItem) {
              lastItem[0].style.display = 'inline-block';
              lastItem[0].style['vertical-align'] = 'middle';
              lastAddElementWidth = lastItem[0].offsetWidth;
            }
            if (inputWidth > 0 && inputWidth - lastAddElementWidth > 0) {
              this.collapseCounter += 1;
            }
            // 隐藏掉超出输入框长度的元素
            if (
              this.collapseCounter === this.selectedVMs.length ||
              this.selectedVMs.length === 1
            )
              return;
            for (
              let i = this.collapseCounter;
              i < this.selectedVMs.length;
              i++
            ) {
              this.$nextTick(() => {
                this.$refs[`item_${i}`][0].style.display = 'none';
              });
            }
          });
        });
      }
    });

    this.$on('click', ({ selectedVM }) => {
      // 相等情况走这里重新设置，其他情况走下面 select;
      if (selectedVM && selectedVM === this.selectedVM && this.filterable) {
        this.filterText = this.selectedVM.currentText;
      }
    });

    this.$on('select', ($event) => {
      if (this.multiple) {
        this.preventBlur = true;
        // 去掉appendTo判断：filterable的多选，会选择后关闭，也需要preventBlur
        this.preventRootBlur = true;
      }

      if (this.filterable) {
        this.filterText = this.selectedVM ? this.selectedVM.currentText : '';
        setTimeout(() => {
          this.$refs.input.focus();
          setTimeout(() => {
            const inputEl = this.$refs.input.$refs.input;
            inputEl.setSelectionRange(
              inputEl.value.length,
              inputEl.value.length,
            );
          });
        });
      }
    });

    // 修复单选，点击已选中值无法自动关闭
    this.$on('click', ($event) => {
      if (!this.multiple) {
        this.close();
      }
    });
    if (this.autoCheckSelectedValue) {
      this.setSelectedDataQueue(this.value);
    }
  },
  mounted() {
    this.autofocus && this.$el.focus();
    // 在编辑器里不要打开
    if (!this.$env.VUE_APP_DESIGNER) this.toggle(this.opened);
    this.setPopperWidth();
  },
  destroyed() {
    clearTimeout(this.inputBlurTimer);
    clearTimeout(this.rootBlurTimer);
    clearTimeout(this.inputDeleteTimer);
    if (this.loadMoreNoopTimer) {
      clearTimeout(this.loadMoreNoopTimer);
    }
  },
  methods: {
    getExtraParams() {
      return { filterText: this.filterText };
    },
    getDataSourceOptions() {
      return {
        viewMode: 'more',
        paging: this.paging,
        remotePaging: this.remotePaging,
        filtering: this.filtering,
        remoteFiltering: this.remoteFiltering,
        getExtraParams: this.getExtraParams,
        // 新增
        sorting: this.sorting,
        remoteSorting: this.remoteSorting,
      };
    },
    normalizeDataSource(dataSource) {
      const options = this.getDataSourceOptions();
      const isNew = typeof this.pagination !== 'undefined';
      const Constructor = isNew ? DataSourceNew : DataSource;

      if (
        dataSource instanceof DataSource ||
        dataSource instanceof DataSourceNew
      )
        return dataSource;
      else if (dataSource instanceof Array) {
        options.data = Array.from(dataSource);
        // 使用了新的分页, 数组类型肯定不是后端数据
        if (isNew) {
          options.remotePaging = false;
          options.remoteFiltering = false;
          options.remoteSorting = false;
        }

                return new Constructor(options);
            } else if (dataSource instanceof Function) {
                const self = this;
                options.load = function load(params, extraParams) {
                    self.$emitSyncParams(params);
                    const result = dataSource(params, extraParams);
                    if (result instanceof Promise)
                        return result.catch(
                            () => (this.currentLoading = false),
                        );
                    else if (result instanceof Array)
                        return Promise.resolve(result);
                    else
                        return Promise.resolve(result);
                };

        // 使用了新的分页, 函数类型先当做是后端数据
        if (isNew) {
          options.remotePaging = !!this.pagination;
          options.remoteFiltering = !!this.filterable;
          options.remoteSorting = !!(this.sorting && this.sorting.field);
        }

        return new Constructor(options);
      } else if (dataSource instanceof Object) {
        if (dataSource.hasOwnProperty('list') && Array.isArray(dataSource.list))
          return new Constructor(
            Object.assign(options, dataSource, {
              data: dataSource.list,
            }),
          );
        return new Constructor(Object.assign(options, dataSource));
      } else return undefined;
    },
    shift(count) {
      let focusedIndex = this.itemVMs.indexOf(
        this.focusedVM || this.selectedVM,
      );
      if (count > 0) {
        for (let i = focusedIndex + count; i < this.itemVMs.length; i++) {
          const itemVM = this.itemVMs[i];
          if (!itemVM.disabled) {
            this.focusedVM = itemVM;
            this.$emit(
              'shift',
              { focusedIndex, focusedVM: itemVM, value: itemVM.value },
              this,
            );
            this.ensureFocusedInView();
            break;
          }
        }
      } else if (count < 0) {
        if (focusedIndex === -1) focusedIndex = this.itemVMs.length;
        for (let i = focusedIndex + count; i >= 0; i--) {
          const itemVM = this.itemVMs[i];
          if (!itemVM.disabled) {
            this.focusedVM = itemVM;
            this.$emit(
              'shift',
              {
                focusedIndex,
                focusedVM: itemVM,
                value: itemVM.value,
              },
              this,
            );
            this.ensureFocusedInView();
            break;
          }
        }
      }
    },
    open() {
      this.$refs.popper && this.$refs.popper.open();
    },
    close() {
      this.$refs.popper && this.$refs.popper.close();
    },
    toggle(opened) {
      if (this.isPreview) return;
      this.$refs.popper && this.$refs.popper.toggle(opened);
    },
    designerControl() {
      this.toggle();
    },
    onOpen($event) {
      // 有时候加载时机问题(v-show 导致的disply:none)，popperWidth会没有获取到，重新获取
      if (this.currentPopperWidth === '0px' || this.appendTo === 'body') {
        this.setPopperWidth();
      }
      this.popperOpened = true; // 刚打开时，除非是没有加载，否则保留上次的 filter 过的数据
      if (
        this.filterable &&
        this.currentDataSource &&
        !this.currentDataSource.initialLoaded
      ) {
        this.load().then(() => {
          this.ensureFocusedInView(true);
          this.$refs.input.focus();
          this.$nextTick(() => this.loadMoreNoopOnOpen());
        });
      } else {
        setTimeout(() => this.ensureFocusedInView(true));
        this.$nextTick(() => this.loadMoreNoopOnOpen());
      }

      this.$emit('open', $event, this);
      this.$emit('update:opened', true);
    },
    onClose($event) {
      if (this.loadMoreNoopTimer) {
        clearTimeout(this.loadMoreNoopTimer);
      }
      this.popperOpened = false;
      this.focusedVM = undefined;
      this.preventRootBlur = false;
      this.preventBlur = false;
      clearTimeout(this.inputBlurTimer);
      clearTimeout(this.rootBlurTimer);
      this.$emit('close', $event, this);
      this.$emit('update:opened', false);
    },
    fastLoad(more, keep) {
      if (!this.currentDataSource) return;
      this.currentDataSource.filter(this.filtering);
      return this.currentDataSource.mustRemote()
        ? this.debouncedLoad(more, keep)
        : this.load(more, keep);
    },
    load(more, keep) {
      const dataSource = this.currentDataSource;
      if (!dataSource) return;
      // if (this.currentLoading)
      //     return Promise.resolve(); // @TODO: dataSource 的多次 promise 必须串行
      // return this.promiseSequence = this.promiseSequence.then(() => {
      if (this.$emitPrevent('before-load', undefined, this)) {
        return;
      }
      this.currentLoading = true;
      const loadToken = (this.loadToken = +new Date());
      // console.log('filterText', this.filterText, loadToken);
      return dataSource[more ? 'loadMore' : 'load']()
        .then((data) => {
          // console.log('loaded', this.loadToken, loadToken);
          if (this.loadToken !== loadToken) return Promise.resolve();
          this.currentLoading = false;
          this.ensureSelectedInItemVMs();
          // 选中数据不在第一页处理
          // 只需要初始的时候处理，这里存储数据用于判断是否是第一次加载数据
          if (this.autoCheckSelectedValue && !this.selectedDataQueue[1]) {
            this.selectedDataQueue.splice(1, 1, data);
          }
          this.$refs.popper.currentOpened && this.$refs.popper.scheduleUpdate();
          this.$emit('load', undefined, this);
          return data;
        })
        .catch(() => (this.currentLoading = false)); // });
    },
    onFocus(e) {
      this.$emit('focus', e, this);
    },
    onInput(value) {
      if (!this.filterable) {
        return;
      }
      this.filterInputFocused = true;
      this.currentText = value; // value.split(',')
      if (this.$emitPrevent('before-filter', { filterText: value }, this))
        return;
      this.filterText = value;
      this.fastLoad(false, true);
      this.open();
      this.hasFilter = true; // 控制blur时是否重置列表，避免每次blur都重置
    },
    onBlur(e) {
      this.filterInputFocused = false;
      if (!this.filterable) {
        return; // 这边必须要用 setTimeout，$nextTick 也不行，需要保证在 @select 之后完成
      }
      this.preventBlur = false;
      this.inputBlurTimer = setTimeout(() => {
        if (this.preventBlur) return (this.preventBlur = false);
        this.selectByText(this.filterText);
        if (this.filterText === '' && !this.selectedVM) {
          this.$emit('blur', e);
        }

        if (this.hasFilter) {
          this.resetFilterList();
          this.hasFilter = false;
        }
      }, 200);
    },
    onRootBlur(e) {
      this.rootBlurTimer = setTimeout(() => {
        if ((this.$refs.input && this.$refs.input.focused) || this.preventBlur)
          return;
        if (this.preventRootBlur) return (this.preventRootBlur = false);
        this.close();
        this.$emit('blur', e);
      }, 400);
    },
    selectByText(text) {
      if (!text) return;
      if (this.multiple) {
        const oldVMs = this.selectedVMs;
        const selectedVM = this.itemVMs.find(
          (itemVM) => itemVM.currentText === text,
        ); // 如果没有匹配项则恢复到上一个状态
        if (!selectedVM && text) {
          if (this.autoComplete) {
            this.prependItem(text);
            this.$nextTick(() => this.select(this.itemVMs[0], true));
          } else {
            this.filterText = '';
            this.currentText = this.filterText;
            this.fastLoad(); // ensure
          }
        } else {
          if (oldVMs.some((itemVM) => itemVM.currentText === text)) {
            this.filterText = '';
            this.fastLoad(); // ensure
            return;
          }
          this.select(selectedVM, true);
        }
      } else {
        const oldVM = this.selectedVM;
        if (!oldVM && !text) return;
        else if (oldVM && text === oldVM.currentText)
          return this.ensureSelectedInItemVMs();
        const selectedVM = this.itemVMs.find(
          (itemVM) => itemVM.currentText === text,
        ); // 如果没有匹配项则恢复到上一个状态
        if (!selectedVM && text) {
          if (this.autoComplete) {
            this.prependItem(text);
            this.$nextTick(() => this.select(this.itemVMs[0], false));
          } else {
            this.filterText = oldVM ? oldVM.currentText : '';
            if (this.filterText === '') {
              this.currentText = this.filterText;
            }
            this.fastLoad(); // ensure
          }
        } else this.select(selectedVM, false);
      }
    },
    prependItem(text) {
      this.currentDataSource.prepend({ text, value: text });
    },
    onEnter(event) {
      // 当footer里的输入框按enter的时候，阻止行为
      if (this.$refs.footer && this.$refs.footer.contains(event.target)) return;
      if (this.focusedVM) this.select(this.focusedVM);
      this.popperOpened ? this.close() : this.open();
    },
    onInputDelete() {
      // 增加setTimeout原因：没有setTimeout，该函数会在onInput前执行，filterText会差一个字母
      // multiple下，第一次删除为空时不希望处理selectedVMs，所以不用放到setTimeout里
      if (!this.multiple) {
        clearTimeout(this.inputDeleteTimer);
        this.inputDeleteTimer = setTimeout(() => {
          if (this.filterable && this.filterText === '') {
            this.selectedVM = undefined; // 清空时清除下拉选中项
            const value = undefined;
            this.$emit('input', value, this);
            this.$emit('update:value', value, this);
          }
        }, 0);
      } else {
        if (this.filterable && this.filterText === '') {
          if (!this.selectedVMs.length) return;
          const lastItemVM = this.selectedVMs[this.selectedVMs.length - 1];
          this.select(lastItemVM, false);
          this.resetFilterList();
        }
      }
    },
    clear() {
      this.preventBlur = true;
      this.currentText = '';
      if (this.multiple) {
        const oldValue = this.value;
        let value = [];
        if (this.converter) {
          value = this.currentConverter.get(value);
        }
        if (this.$emitPrevent('before-clear', { oldValue, value }, this))
          return;
        this.selectedVMs.forEach((itemVM) => (itemVM.currentSelected = false));
        this.selectedVMs = [];
        this.filterText = '';
        this.fastLoad();
        this.$emit('input', value, this);
        this.$emit('update:value', value, this);
        this.$emit('clear', { oldValue, value }, this);
      } else {
        const oldValue = this.value;
        let value;
        if (this.converter) {
          value = this.currentConverter.get(value);
        }
        if (this.$emitPrevent('before-clear', { oldValue, value }, this))
          return;
        this.selectedVM = undefined;
        this.filterText = '';
        this.fastLoad();
        this.$emit('input', this.handleEmptyValue(value), this);
        this.$emit('update:value', this.handleEmptyValue( value ), this);
        this.$emit('clear', { oldValue, value:this.handleEmptyValue( value ) }, this);
      }
      this.focus();
      this.close();
    },
    focus() {
      this.preventBlur = true;
      if (this.filterable) {
        this.$refs.input.focus();
      } else {
        this.$emit('focus');
      }
    },
    blur() {
      if (this.filterable) {
        this.$refs.input.blur();
      } else {
        this.$emit('blur');
      }
    },
    setPopperWidth() {
      if (this.appendTo === 'body') {
        this.currentPopperWidth = this.popperWidth
          ? this.popperWidth
          : this.$el && this.$el.offsetWidth + 'px';
      } else {
        this.currentPopperWidth = this.popperWidth || '100%';
      }
    },
    rootFocus() {
      this.$el.focus();
    },
    resetFilterList() {
      if (this.multiple) {
        this.fastLoad();
      }
    },
    removeTag(itemVm, flag) {
      this.select(itemVm, flag);
      this.resetFilterList();
      this.$emit('removetag', itemVm);
      this.preventRootBlur = false;
      this.preventBlur = false;
      this.rootFocus();
    },
    onScroll(e) {
      this.hasScroll = true;
      this.throttledVirtualScroll(e);

      if (typeof this.pagination !== 'undefined') {
        if (!this.pagination || !this.$options.isSelect) {
          return;
        }
      } else {
        if (
          !(
            this.pageable === 'auto-more' ||
            (this.pageable === true && this.$options.isSelect)
          )
        ) {
          return;
        }
      }

      if (this.currentLoading) {
        return;
      }

      const el = e.target;
      if (
        Math.abs(el.scrollHeight - (el.scrollTop + el.clientHeight)) <= 1 &&
        this.currentDataSource &&
        this.currentDataSource.hasMore()
      )
        this.debouncedLoad(true);
    },
    /**
     * 配合showRenderFooter，添加项时调用
     */
    addItem(item, inFirst) {
      if (inFirst) {
        this.currentData.unshift(item);
      } else {
        this.currentData.push(item);
      }
      this.$forceUpdate();
      this.$nextTick(() => {
        const index = inFirst ? 0 : this.currentData.length - 1;
        this.itemScrollIntoView(index);
      });
    },
    itemScrollIntoView(index) {
      const children = this.$refs.popperwrap.children;
      if (children[index]) {
        children[index].scrollIntoView({
          behavior: 'smooth',
          block: 'nearest',
          inline: 'nearest',
        });
      }
    },
    onDelete(event) {
      // 当footer里的输入框按delete的时候，阻止行为，不然弹层会关闭
      if (this.$refs.footer && this.$refs.footer.contains(event.target)) return;
      if (this.clearable) {
        this.clear();
      }
    },
    /**
     * 当下拉列表是分页的而传入的value值不在第一页的情况下，选项没有选中展示
     * 判断是否在列表里，没在列表里会开启循环加载数据
     */
    async loadMoreForSelected() {
      const data = this.currentDataSource.viewData;
      const value = this.selectedDataQueue[0];
      if (value && data) {
        let allInData = false;
        if (Array.isArray(value)) {
          allInData = value.every(
            (val) =>
              !!data.find(
                (item) => '' + this.$at2(item, this.valueField) === '' + val,
              ),
          );
        } else {
          allInData = !!data.find(
            (item) => '' + this.$at2(item, this.valueField) === '' + value,
          );
        }
        const currentDataSource = this.currentDataSource;
        if (!allInData && currentDataSource && currentDataSource.hasMore()) {
          await currentDataSource.loadMore();
          this.loadMoreForSelected();
        }
      }
    },
    /**
     * 存储value
     */
    setSelectedDataQueue(value) {
      // 添加配置，可关闭自动查找
      if (!this.autoCheckSelectedValue) return;
      if (this.multiple) {
        let currentValue = value;
        if (this.converter) currentValue = this.currentConverter.set(value);
        if (Array.isArray(currentValue)) {
          this.selectedDataQueue.splice(0, 1, currentValue);
        }
      } else {
        this.selectedDataQueue.splice(0, 1, value);
      }
    },
    async loadMoreNoopOnOpen() {
      if (!this.usePagination || !this.$refs.popperwrap) {
        return;
      }

      const scrollerEl = this.showRenderFooter
        ? this.$refs.popperwrap
        : this.$refs.popper.$el;
      if (!scrollerEl) {
        return;
      }

      // 有滚动条等滚动加载
      if (scrollerEl.scrollHeight > scrollerEl.offsetHeight) {
        return;
      }

      if (this.loadMoreNoopTimer) {
        clearTimeout(this.loadMoreNoopTimer);
      }

      const data = await this.load(true);
      if (!data || data.length === 0) {
        return;
      }

      /**
       * 渲染结束后加载下一页
       */
      this.loadMoreNoopTimer = setTimeout(() => this.loadMoreNoopOnOpen(), 50);
    },
    handleEmptyValue(value) {
      if (!this.emptyValueIsNull) {
        return value;
      } else {
        return value ? value : null;
      }
    }
  },
};
</script>

<style module src="./index.css"></style>
