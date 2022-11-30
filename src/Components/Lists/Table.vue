<script setup>
import { computed, onMounted, ref, unref, watch } from 'vue';
import { ChevronDownIcon, ChevronUpIcon } from '@heroicons/vue/20/solid';
import Pagination from '../Navigation/Pagination/Pagination';
import Spinner from '../Elements/Spinner';

const props = defineProps({
  fields: Array,
  filter: String,
  filters: {
    type: Object,
    default() {
      return ref({});
    }
  },
  items: {
    type: [Array, Function],
    required: true,
  },
  perPage: {
    type: Number,
    default: 0,
    validator(value) {
      return value >= 0;
    }
  },
  responsive: {
    type: Boolean,
    default: false,
  },
  sortBy: String,
  sortDir: {
    type: String,
    default: 'asc',
  },
  striped: Boolean,
  tbodyTdClass: [ Array, Function, String ],
  tbodyTrClass: [ Array, Function, String ],
});

const emit = defineEmits(['sort', 'update:sortBy']);

const tableData = ref([]);
const currentPage = ref(1);
const currentSortBy = ref(props.sortBy);
const currentSortDir = ref(props.sortDir);
const busy = ref(false);
const totalRows = ref(_.isArray(props.items) ? props.items.length : 0);

const hasItems = computed(() => unref(tableData).length > 0);
const tableFields = computed(() => {
  if (!props.fields || _.isEmpty(props.fields)) {
    return _.chain(unref(tableData)).first().map((value, index) => {
      return {
        label: _.startCase(index),
        key: index,
      };
    }).value();
  }

  return _.chain(props.fields)
      .map((field) => {
        if (!field) {
          return;
        }

    if (_.isObject(field)) {
      return {...field, label: getLabel(field), key: field.key || _.snakeCase(field.label) };
    }

    return { label: getLabel(field), key: field };
      })
      .filter()
      .value();
});
const requestContext = computed(() => {
  return {
    page: unref(currentPage),
    filter: props.filter || undefined,
    filters: props.filters || undefined,
    perPage: props.perPage || undefined,
    sortBy: unref(currentSortBy),
    sortDir: unref(currentSortDir) || 'asc',
  };
});

const getLabel = (field) => _.isObject(field) ? field.label ?? _.startCase(field.key) : `${field}`;
const getValue = (item, key) => _.get(item, key);
const isSortable = (field) => _.isObject(field) && (field?.sortable === true);
const isSortedBy = (field) => unref(currentSortBy) === field;
const fetch = async () => {
  if (busy.value) {
    return Promise.reject('Busy');
  }

  busy.value = true;

  if (_.isFunction(props.items)) {
    const response = props.items(requestContext.value, setData);

    if (_.isNull(response) || _.isUndefined(response)) {
      busy.value = false;
      return Promise.resolve(response);
    }

    return Promise
        .resolve(response)
        .then((value) => {
          setData(value);

          return value;
        })
        .finally(() => busy.value = false);
  }

  if (_.isObject(props.items) || _.isArray(props.items)) {
    // clone the items and sort them
    const items = _.chain(props.items)
        .castArray()
        .cloneDeep()
        .sortBy((o) => o[currentSortBy.value])
        .thru((value) => currentSortDir.value === 'desc' ? value.reverse() : value)
        .value();

    setData(items);
  } else {
    setData([]);
  }

  busy.value = false;
  return Promise.resolve(unref(tableData));
};
const onPaginate = (page) => {
  currentPage.value = page;

  fetch();
};
const setData = (data) => {
  if (!_.isArray(data) && _.isObject(data)) {
    tableData.value = data.data?.data || data.data || [];
    totalRows.value = data.data?.total || data.total || tableData.value.length;
  } else {
    tableData.value = data || [];
    totalRows.value = tableData.value.length;
  }
};
const sortData = (field, direction) => {
  if (direction) {
    toggleDir(direction);
  } else if (unref(currentSortBy) !== field) {
    toggleDir('asc');
  } else {
    toggleDir();
  }

  currentSortBy.value = field;

  fetch();
};
const toggleDir = (direction) => {
  if (direction) {
    currentSortDir.value = _.toLower(direction);
  } else {
    switch (_.toLower(unref(currentSortDir))) {
      case 'asc':
        currentSortDir.value = 'desc';
        break;
      case 'desc':
        currentSortDir.value = 'asc';
        break;
    }
  }

  emit('update:sortBy', unref(currentSortDir));
};
const parseStyles = (value, item) => {
  if (_.isFunction(value)) {
    return value(item);
  }

  return value;
};
const toggleDetails = (index) => {
  if (tableData.value[index]) {
    tableData.value[index]._showDetails = !tableData.value[index]._showDetails;
  }
};

defineExpose({
  fetch,
  sortData,
});

watch(
    [props.filters, () => _.isFunction(props.items) ? null : props.items],
    async (newFilters, oldFilters) => {
      return fetch();
    }
);

onMounted(() => {
  fetch();
});
</script>

<template>
  <div v-if="hasItems || busy">
    <div :class="responsive ? 'overflow-x-auto' : undefined">
      <div :class="[responsive ? 'inline-block min-w-full py-2 align-middle md:px-1' : undefined]">
        <div class="overflow-hidden shadow ring-1 ring-black ring-opacity-5 md:rounded dark:shadow-slate-800 dark:ring-slate-700">
          <table class="min-w-full divide-y divide-gray-300 dark:divide-slate-500">
            <thead class="bg-gray-50 dark:bg-gray-800">
              <tr>
                <th
                    v-for="field in tableFields"
                    scope="col"
                    class="py-3.5 px-3 text-left text-sm font-semibold text-gray-900 dark:text-gray-300"
                    :class="field.class"
                >
                  <button v-if="isSortable(field)" class="group inline-flex" @click="sortData(field.key)">
                    {{ field.label }}

                    <span
                        class="ml-2 flex-none rounded"
                        :class="isSortedBy(field.key) ? 'bg-gray-200 text-gray-900 group-hover:bg-gray-300 dark:bg-slate-600 dark:group-hover:bg-slate-500' : 'invisible text-gray-400 group-hover:visible group-focus:visible'"
                    >
                      <component
                          :is="isSortedBy(field.key) && currentSortDir === 'desc' ? ChevronUpIcon : ChevronDownIcon"
                          class="h-5 w-5" aria-hidden="true"
                        />
                    </span>
                  </button>
                  <template v-else>
                    {{ field.label }}
                  </template>
                </th>
              </tr>
            </thead>

            <tbody class="divide-y divide-gray-200 bg-white dark:divide-slate-600 dark:bg-slate-800">
              <tr v-if="busy" class="bg-gray-50 dark:bg-slate-700">
                <td :colspan="tableFields.length || 1">
                  <slot name="busy">
                    <div class="text-center my-4">
                      <h3 class="mt-2 text-lg font-medium">Loading...</h3>

                      <Spinner class="mt-4 w-8 h-8 mx-auto" />
                    </div>
                  </slot>
                </td>
              </tr>

            <template
                  v-else
                  v-for="(item, index) in tableData"
            >
              <tr
                  :class="[
                      'text-sm text-gray-500 dark:text-slate-400',
                      striped && index % 2 === 1 ? 'bg-gray-50 dark:bg-slate-700' : undefined,
                      parseStyles(tbodyTrClass, item)
                  ]"
              >
                <td
                    v-for="(field) in tableFields"
                    :class="[
                        'whitespace-nowrap px-3 py-4',
                        parseStyles(field.class, item),
                        parseStyles(field.bodyClass, item),
                        parseStyles(tbodyTdClass, item),
                    ]"
                >
                  <slot
                      :name="`cell(${field.key})`"
                      :value="getValue(item, field.key)"
                      :item="item"
                      :index="index"
                      :toggleDetails="() => toggleDetails(index)"
                      :detailsShowing="item._showDetails || false"
                  >
                    <slot
                        name="cell"
                        :value="getValue(item, field.key)"
                        :item="item"
                        :index="index"
                        :toggleDetails="() => toggleDetails(index)"
                        :detailsShowing="item._showDetails || false"
                    >
                      {{ getValue(item, field.key) || '' }}
                    </slot>
                  </slot>
                </td>
              </tr>

              <template v-if="$slots['row-details'] && item._showDetails">
                <tr>
                  <td :colspan="tableFields.length" class="-mt-px px-3 py-4">
                    <slot
                        name="row-details"
                        :item="item"
                        :index="index"
                        :fields="tableFields"
                        :toggleDetails="() => toggleDetails(index)"
                        :detailsShowing="item._showDetails || false"
                    />
                  </td>
                </tr>
              </template>
            </template>
            </tbody>
          </table>
        </div>
      </div>
    </div>

    <Pagination
        v-if="perPage > 0"
        :per-page="perPage"
        :total-rows="totalRows"
        :value="currentPage"
        :disabled="busy"
        @change="onPaginate"
        class="mt-4"
    />
  </div>

  <div v-else>
    <slot name="empty" />
  </div>
</template>
