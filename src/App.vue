<script setup>
import { ref, computed, onMounted, nextTick } from "vue";
const initial = [
  { id: "insurance", name: "保險推廣" },
  { id: "safety", name: "職災防禦" },
  { id: "partners", name: "合作廠商" },
];
const tabs = ref(initial),
  active = ref("insurance"),
  draft = ref(""),
  error = ref(""),
  notice = ref(""),
  selected = ref([]),
  descending = ref(true),
  menu = ref(false),
  mobile = ref(false);
const rows = ref([
  {
    id: "1",
    tabId: "insurance",
    title:
      "🎬 短片美學創作及品牌塑造實務班 🔔 9/17(四)中午12:00台灣就業通開放報名！",
    date: "2026/09/01 10:19:42",
    image: 0,
  },
  {
    id: "2",
    tabId: "insurance",
    title: "🎬 片場安全必備技能實務課｜開放報名！",
    date: "2026/08/18 15:35:43",
    image: 1,
  },
  {
    id: "3",
    tabId: "insurance",
    title: "【自辦班】🔔 建立友善職場｜職場霸凌防治實務解析",
    date: "2026/08/18 15:35:43",
    image: 2,
  },
  {
    id: "4",
    tabId: "insurance",
    title: "📢【免費報名】115年度中高齡及高齡工作者安全衛生宣導會",
    date: "2026/07/16 14:35:41",
    image: 3,
  },
]);
const current = computed(() => tabs.value.find((t) => t.id === active.value));
const visible = computed(() =>
  rows.value
    .filter((r) => r.tabId === active.value)
    .sort((a, b) =>
      descending.value
        ? b.date.localeCompare(a.date)
        : a.date.localeCompare(b.date),
    ),
);
const dialog = ref(null),
  mode = ref("rename"),
  rename = ref(""),
  dialogError = ref(""),
  deleteName = ref(""),
  management = ref(null);
const storageKey = "movieu-tab-admin-v1";
onMounted(() => {
  try {
    const d = JSON.parse(localStorage.getItem(storageKey) || "null");
    if (
      d &&
      Array.isArray(d.tabs) &&
      d.tabs.every(
        (t) => typeof t.id === "string" && typeof t.name === "string",
      ) &&
      Array.isArray(d.rows) &&
      d.rows.every(
        (r) =>
          typeof r.id === "string" &&
          typeof r.tabId === "string" &&
          typeof r.date === "string" &&
          typeof r.title === "string",
      )
    ) {
      tabs.value = d.tabs;
      rows.value = d.rows;
      active.value = d.tabs.some((t) => t.id === d.active)
        ? d.active
        : d.tabs[0]?.id || "";
    }
  } catch {
    notice.value = "無法讀取儲存內容，已使用初始資料。";
  }
});
function persist(message) {
  notice.value = message;
  try {
    localStorage.setItem(
      storageKey,
      JSON.stringify({
        tabs: tabs.value,
        rows: rows.value,
        active: active.value,
      }),
    );
  } catch {
    notice.value = message + "；瀏覽器無法儲存，重新整理後可能遺失。";
  }
}
function validate(name, except = "") {
  if (!name.trim()) return "請輸入頁籤名稱";
  if ([...name.trim()].length > 20) return "名稱最多 20 個字";
  if (tabs.value.some((t) => t.id !== except && t.name === name.trim()))
    return "此頁籤名稱已存在";
  return "";
}
function add() {
  error.value = validate(draft.value);
  if (error.value) return;
  const t = { id: crypto.randomUUID(), name: draft.value.trim() };
  tabs.value.push(t);
  active.value = t.id;
  draft.value = "";
  selected.value = [];
  menu.value = false;
  persist("已新增分頁。");
}
function choose(id) {
  active.value = id;
  selected.value = [];
  menu.value = false;
  persist("");
}
async function open(action) {
  mode.value = action;
  rename.value = current.value.name;
  deleteName.value = "";
  dialogError.value = "";
  menu.value = false;
  dialog.value.showModal();
  await nextTick();
  dialog.value.querySelector("input")?.focus();
}
function close() {
  dialog.value.close();
  nextTick(() => management.value?.focus());
}
function save() {
  if (mode.value === "rename") {
    dialogError.value = validate(rename.value, active.value);
    if (dialogError.value) return;
    current.value.name = rename.value.trim();
    persist("頁面名稱及頁籤名稱已同步更新。");
  } else {
    if (deleteName.value !== current.value.name) {
      dialogError.value = "請輸入完整分頁名稱以確認刪除";
      return;
    }
    const old = active.value,
      index = tabs.value.findIndex((t) => t.id === old);
    tabs.value = tabs.value.filter((t) => t.id !== old);
    rows.value = rows.value.filter((r) => r.tabId !== old);
    active.value = tabs.value[Math.min(index, tabs.value.length - 1)]?.id || "";
    selected.value = [];
    persist("已刪除分頁與其示範內容。");
  }
  close();
}
</script>

<template>
  <div class="layout" @keydown.esc="menu = false">
    <aside :class="{ mobile }">
      <div class="brand" role="img" aria-label="台北市電影戲劇業職業工會"></div>
      <nav aria-label="主選單">
        <button disabled><span>⌂</span>官網設定</button>
        <button class="active" aria-current="page" @click="mobile = false">
          <span>⊕</span>新增主選單
        </button>
        <button
          v-for="(n, i) in [
            '最新消息',
            '下載區',
            '常見問題',
            '活動專區',
            '人員資料庫',
          ]"
          :key="n"
          disabled
        >
          <span>{{ ["◁", "⇩", "?", "▧", "▧"][i] }}</span
          >{{ n }}
        </button>
      </nav>
    </aside>
    <main>
      <button
        class="mobile-toggle"
        @click="mobile = !mobile"
        :aria-expanded="mobile"
      >
        ☰ 主選單
      </button>
      <div class="toolbar">
        <div
          class="tabs"
          role="tablist"
          aria-label="分頁"
          @keydown.right.prevent="
            choose(
              tabs[(tabs.findIndex((t) => t.id === active) + 1) % tabs.length]
                ?.id || '',
            )
          "
          @keydown.left.prevent="
            choose(
              tabs[
                (tabs.findIndex((t) => t.id === active) - 1 + tabs.length) %
                  tabs.length
              ]?.id || '',
            )
          "
        >
          <button
            v-for="t in tabs"
            :key="t.id"
            role="tab"
            :aria-selected="active === t.id"
            :class="{ primary: active === t.id }"
            @click="choose(t.id)"
          >
            {{ t.name }}
          </button>
        </div>
        <form class="add-form" @submit.prevent="add">
          <input
            v-model="draft"
            aria-label="新頁籤名稱"
            placeholder="請輸入頁籤名稱"
            :aria-invalid="!!error"
            aria-describedby="add-error"
            @input="error = ''"
          /><button class="primary">新增</button
          ><button
            type="button"
            @click="
              draft = '';
              error = '';
            "
          >
            取消
          </button>
        </form>
      </div>
      <p id="add-error" class="error" v-if="error" role="alert">{{ error }}</p>
      <p class="notice" v-if="notice" role="status">{{ notice }}</p>
      <section class="panel" v-if="current">
        <div class="panel-heading">
          <div class="heading-group">
            <h1>{{ current.name }}</h1>
            <div
              class="manage"
              @focusout="
                (e) => {
                  if (!e.currentTarget.contains(e.relatedTarget)) menu = false;
                }
              "
            >
              <button
                ref="management"
                :aria-expanded="menu"
                @click="menu = !menu"
              >
                管理 ▾
              </button>
              <div class="dropdown" v-if="menu">
                <button @click="open('rename')">✎　修改頁面名稱</button
                ><button class="danger-text" @click="open('delete')">
                  ♧　刪除整個分頁
                </button>
              </div>
            </div>
          </div>
          <button
            class="sort"
            @click="descending = !descending"
            :aria-label="descending ? '改為日期升冪' : '改為日期降冪'"
          >
            {{ descending ? "↓" : "↑" }} 排序
          </button>
        </div>
        <div class="table-scroll">
          <table v-if="visible.length">
            <thead>
              <tr>
                <th class="check"><span class="sr-only">選取</span></th>
                <th>縮圖／標題</th>
                <th class="date">日期</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="r in visible" :key="r.id">
                <td>
                  <input
                    type="checkbox"
                    v-model="selected"
                    :value="r.id"
                    :aria-label="`選取 ${r.title}`"
                  />
                </td>
                <td>
                  <div class="article">
                    <div
                      class="thumb"
                      :style="{
                        '--image-y': `${[-324, -465, -610, -750][r.image] || -324}px`,
                      }"
                      role="img"
                      :aria-label="r.title"
                    ></div>
                    <span>{{ r.title }}</span>
                  </div>
                </td>
                <td class="date">{{ r.date }}</td>
              </tr>
            </tbody>
          </table>
        </div>
        <div v-if="!visible.length" class="empty">此分頁尚無內容</div>
        <footer>
          <span v-if="selected.length">已選取 {{ selected.length }} 筆</span
          ><span>共 {{ visible.length }} 筆</span
          ><button v-if="visible.length" class="primary" aria-current="page">
            1
          </button>
        </footer>
      </section>
      <section v-else class="panel empty">尚無分頁，請在上方新增頁籤。</section>
    </main>
    <dialog ref="dialog" @cancel="menu = false">
      <form @submit.prevent="save">
        <h2>{{ mode === "rename" ? "修改頁面名稱" : "刪除整個分頁" }}</h2>
        <template v-if="mode === 'rename'"
          ><label for="rename">頁面名稱</label
          ><input id="rename" v-model="rename" />
          <p>儲存後會同步更新上方頁籤名稱。</p></template
        ><template v-else
          ><p>
            確定刪除「{{ current?.name }}」？此操作會一併移除該分頁的
            {{ visible.length }} 筆示範內容，無法復原。
          </p>
          <label for="delete-name">請輸入「{{ current?.name }}」確認</label
          ><input id="delete-name" v-model="deleteName" autocomplete="off"
        /></template>
        <p class="error" role="alert" v-if="dialogError">{{ dialogError }}</p>
        <div class="dialog-actions">
          <button type="button" @click="close">取消</button
          ><button :class="mode === 'delete' ? 'danger' : 'primary'">
            {{ mode === "rename" ? "儲存" : "確認刪除" }}
          </button>
        </div>
      </form>
    </dialog>
  </div>
</template>
