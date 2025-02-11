<template>
  <pw-section class="green" icon="history" label="History" ref="history">
    <ul>
      <li id="filter-history">
        <input
          aria-label="Search"
          type="text"
          placeholder="search history"
          v-model="filterText"
        />
      </li>
    </ul>
    <ul v-if="history.length !== 0" class="labels">
      <div class="show-on-small-screen">
        <li>
          <button class="icon">
            <i class="material-icons">star_half</i>
          </button>
        </li>
        <li>
          <button class="icon">
            <i class="material-icons">history</i>
          </button>
        </li>
      </div>
      <li>
        <button class="icon" @click="sort_by_label()">
          Label
        </button>
      </li>
      <li>
        <button class="icon" @click="sort_by_time()">
          Time
        </button>
      </li>
      <li>
        <button class="icon" @click="sort_by_status_code()">
          Status
        </button>
      </li>
      <li>
        <button class="icon" @click="sort_by_url()">
          URL
        </button>
      </li>
      <li>
        <button class="icon" @click="sort_by_path()">
          Path
        </button>
      </li>
      <transition name="smooth" v-if="show">
        <li>
          <ul>
            <li>
              <button class="icon" @click="sort_by_duration()">
                Duration (ms)
              </button>
            </li>
            <li>
              <button class="icon">
                Pre-script
              </button>
            </li>
          </ul>
        </li>
      </transition>
      <div class="show-on-small-screen">
        <li>
          <button
            class="icon"
            @click="enableHistoryClearing"
            v-tooltip="'Clear History'"
          >
            <i class="material-icons">clear_all</i>
          </button>
        </li>
        <li>
          <button
            class="icon"
            @click="toggleCollapse()"
            v-tooltip="{ content: !show ? 'Show more' : 'Hide more' }"
          >
            <i class="material-icons" v-if="!show">first_page</i>
            <i class="material-icons" v-else>last_page</i>
          </button>
        </li>
      </div>
    </ul>
    <virtual-list
      class="virtual-list"
      :class="{ filled: filteredHistory.length }"
      :size="56"
      :remain="Math.min(5, filteredHistory.length)"
    >
      <ul v-for="(entry, index) in filteredHistory" :key="index" class="entry">
        <div class="show-on-small-screen">
          <li>
            <button
              class="icon"
              :class="{ stared: entry.star }"
              @click="toggleStar(index)"
              v-tooltip="{ content: !entry.star ? 'Add star' : 'Remove star' }"
            >
              <i class="material-icons" v-if="entry.star">star</i>
              <i class="material-icons" v-else>star_border</i>
            </button>
          </li>
          <li>
            <button
              class="icon"
              v-tooltip="{
                content: !entry.usesScripts
                  ? 'No pre-request script'
                  : 'Used pre-request script'
              }"
            >
              <i class="material-icons" v-if="!entry.usesScripts">http</i>
              <i class="material-icons" v-else>code</i>
            </button>
          </li>
        </div>
        <div class="show-on-small-screen">
          <li>
            <input
              aria-label="Label"
              type="text"
              readonly
              :value="entry.label"
              placeholder="No label"
            />
          </li>
          <li>
            <input
              aria-label="Time"
              type="text"
              readonly
              :value="entry.time"
              v-tooltip="entry.date"
            />
          </li>
        </div>
        <li class="method-list-item">
          <input
            aria-label="Method"
            type="text"
            readonly
            :value="entry.method"
            :class="findEntryStatus(entry).className"
            :style="{ '--status-code': entry.status }"
          />
          <span
            class="entry-status-code"
            :class="findEntryStatus(entry).className"
            :style="{ '--status-code': entry.status }"
            >{{ entry.status }}</span
          >
        </li>
        <div class="show-on-small-screen">
          <li>
            <input
              aria-label="URL"
              type="text"
              readonly
              :value="entry.url"
              placeholder="No URL"
            />
          </li>
          <li>
            <input
              aria-label="Path"
              type="text"
              readonly
              :value="entry.path"
              placeholder="No path"
            />
          </li>
        </div>
        <transition name="smooth">
          <div v-if="show" class="show-on-small-screen">
            <li>
              <input
                aria-label="Duration"
                type="text"
                readonly
                :value="entry.duration"
                placeholder="No duration"
              />
            </li>
            <li>
              <input
                aria-label="Pre Request Script"
                type="text"
                readonly
                :value="entry.preRequestScript"
                placeholder="No pre request script"
              />
            </li>
          </div>
        </transition>
        <div class="show-on-small-screen">
          <li>
            <button
              v-tooltip="'Delete entry'"
              class="icon"
              :id="'delete-button#' + index"
              @click="deleteHistory(entry)"
              aria-label="Delete"
            >
              <i class="material-icons">delete</i>
            </button>
          </li>
          <li>
            <button
              v-tooltip="'Edit entry'"
              class="icon"
              :id="'use-button#' + index"
              @click="useHistory(entry)"
              aria-label="Edit"
            >
              <i class="material-icons">edit</i>
            </button>
          </li>
        </div>
      </ul>
    </virtual-list>
    <ul
      :class="{ hidden: filteredHistory.length != 0 || history.length === 0 }"
    >
      <li>
        <label>Nothing found "{{ filterText }}"</label>
      </li>
    </ul>
    <ul v-if="history.length === 0">
      <li>
        <label>History is empty</label>
      </li>
    </ul>
    <ul v-if="history.length !== 0">
      <li v-if="!isClearingHistory">
        <button
          class="icon"
          id="clear-history-button"
          :disabled="history.length === 0"
          @click="enableHistoryClearing"
        >
          <i class="material-icons">clear_all</i>
          <span>Clear All</span>
        </button>
      </li>
      <li v-else>
        <div class="flex-wrap">
          <label for="clear-history-button">Are you sure?</label>
          <div>
            <button
              class="icon"
              id="confirm-clear-history-button"
              @click="clearHistory"
            >
              Yes
            </button>
            <button
              class="icon"
              id="reject-clear-history-button"
              @click="disableHistoryClearing"
            >
              No
            </button>
          </div>
        </div>
      </li>
    </ul>
  </pw-section>
</template>

<style scoped lang="scss">
.virtual-list {
  min-height: 90px;

  [readonly] {
    cursor: default;
  }
}

label {
  &:hover {
    cursor: pointer;
    color: var(--fg-color);
  }
}

.smooth-enter-active,
.smooth-leave-active {
  transition: all 0.2s;
}

.smooth-enter,
.smooth-leave-to {
  opacity: 0;
}

.stared {
  color: #f8e81c !important;
}

@media (max-width: 720px) {
  .virtual-list.filled {
    min-height: 320px;
  }

  .labels {
    display: none;
  }

  .entry {
    border-bottom: 1px solid var(--brd-color);
  }
}
</style>

<script>
import VirtualList from "vue-virtual-scroll-list";
import section from "./section";
import { findStatusGroup } from "../pages/index";

const updateOnLocalStorage = (propertyName, property) =>
  window.localStorage.setItem(propertyName, JSON.stringify(property));
export default {
  components: {
    "pw-section": section,
    VirtualList
  },
  data() {
    const localStorageHistory = JSON.parse(
      window.localStorage.getItem("history")
    );
    return {
      history: localStorageHistory || [],
      filterText: "",
      showFilter: false,
      isClearingHistory: false,
      reverse_sort_label: false,
      reverse_sort_time: false,
      reverse_sort_status_code: false,
      reverse_sort_url: false,
      reverse_sort_path: false,
      show: false
    };
  },
  computed: {
    filteredHistory() {
      return this.history.filter(entry => {
        const filterText = this.filterText.toLowerCase();
        return Object.keys(entry).some(key => {
          let value = entry[key];
          value = typeof value !== "string" ? value.toString() : value;
          return value.toLowerCase().includes(filterText);
        });
      });
    }
  },
  methods: {
    clearHistory() {
      this.history = [];
      this.filterText = "";
      this.disableHistoryClearing();
      updateOnLocalStorage("history", this.history);
      this.$toast.error("History Deleted", {
        icon: "delete"
      });
    },
    useHistory(entry) {
      this.$emit("useHistory", entry);
    },
    findEntryStatus(entry) {
      const foundStatusGroup = findStatusGroup(entry.status);
      return (
        foundStatusGroup || {
          className: ""
        }
      );
    },
    deleteHistory(entry) {
      this.history.splice(this.history.indexOf(entry), 1);
      if (this.history.length === 0) {
        this.filterText = "";
      }
      updateOnLocalStorage("history", this.history);
      this.$toast.error("Deleted", {
        icon: "delete"
      });
    },
    addEntry(entry) {
      this.history.push(entry);
      updateOnLocalStorage("history", this.history);
    },
    enableHistoryClearing() {
      if (!this.history || !this.history.length) return;
      this.isClearingHistory = true;
    },
    disableHistoryClearing() {
      this.isClearingHistory = false;
    },
    sort_by_time() {
      let byDate = this.history.slice(0);
      byDate.sort((a, b) => {
        let date_a = a.date.split("/");
        let date_b = b.date.split("/");
        let time_a = a.time.split(":");
        let time_b = b.time.split(":");
        let final_a = new Date(
          date_a[2],
          date_a[1],
          date_a[0],
          time_a[0],
          time_a[1],
          time_a[2]
        );
        let final_b = new Date(
          date_b[2],
          date_b[1],
          date_b[0],
          time_b[0],
          time_b[1],
          time_b[2]
        );
        if (this.reverse_sort_time) return final_b - final_a;
        else return final_a - final_b;
      });
      this.history = byDate;
      this.reverse_sort_time = !this.reverse_sort_time;
    },
    sort_by_status_code() {
      let byCode = this.history.slice(0);
      byCode.sort((a, b) => {
        if (this.reverse_sort_status_code) return b.status - a.status;
        else return a.status - b.status;
      });
      this.history = byCode;
      this.reverse_sort_status_code = !this.reverse_sort_status_code;
    },
    sort_by_url() {
      let byUrl = this.history.slice(0);
      byUrl.sort((a, b) => {
        if (this.reverse_sort_url)
          return a.url == b.url ? 0 : +(a.url < b.url) || -1;
        else return a.url == b.url ? 0 : +(a.url > b.url) || -1;
      });
      this.history = byUrl;
      this.reverse_sort_url = !this.reverse_sort_url;
    },
    sort_by_label() {
      let byLabel = this.history.slice(0);
      byLabel.sort((a, b) => {
        if (this.reverse_sort_label)
          return a.label == b.label ? 0 : +(a.label < b.label) || -1;
        else return a.label == b.label ? 0 : +(a.label > b.label) || -1;
      });
      this.history = byLabel;
      this.reverse_sort_label = !this.reverse_sort_label;
    },
    sort_by_path() {
      let byPath = this.history.slice(0);
      byPath.sort((a, b) => {
        if (this.reverse_sort_path)
          return a.path == b.path ? 0 : +(a.path < b.path) || -1;
        else return a.path == b.path ? 0 : +(a.path > b.path) || -1;
      });
      this.history = byPath;
      this.reverse_sort_path = !this.reverse_sort_path;
    },
    sort_by_duration() {
      let byDuration = this.history.slice(0);
      byDuration.sort((a, b) => {
        if (this.reverse_sort_duration)
          return a.duration == b.duration
            ? 0
            : +(a.duration < b.duration) || -1;
        else
          return a.duration == b.duration
            ? 0
            : +(a.duration > b.duration) || -1;
      });
      this.history = byDuration;
      this.reverse_sort_duration = !this.reverse_sort_duration;
    },
    toggleCollapse() {
      this.show = !this.show;
    },
    toggleStar(index) {
      this.history[index]["star"] = !this.history[index]["star"];
      updateOnLocalStorage("history", this.history);
    }
  }
};
</script>
