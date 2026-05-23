<template>
  <div class="emotional-diary-page">
    <div class="page-container">
      <div class="hero-section">
        <div class="breathing-circle">
          <el-image :src="like" fit="cover" style="width: 40px; height: 40px"></el-image>
        </div>
        <h1 class="page-title">情绪日记</h1>
        <p class="page-desc">记录每日情绪变化，了解自己的内心世界</p>
      </div>

      <div class="diary-form-section">
        <div class="form-card" id="diary-form-card">
          <div class="card-header">
            <h2>{{ isEditing ? '编辑日记' : '今日心情' }}</h2>
            <div class="current-date">{{ displayDate }}</div>
          </div>

          <!-- 情绪评分 -->
          <div class="mood-section">
            <label class="section-label">选择今天的情绪</label>
            <div class="mood-options">
              <div
                v-for="(mood, index) in moodOptions"
                :key="mood.value"
                class="mood-item"
                :class="{ active: currentMood === mood.value }"
                @click="selectMood(mood.value)"
              >
                <span class="mood-emoji">{{ mood.emoji }}</span>
                <span class="mood-label">{{ mood.label }}</span>
              </div>
            </div>
            <el-rate
              v-model="moodScore"
              :max="10"
              show-score
              text-color="#fb923c"
              class="mood-rating"
            />
          </div>

          <!-- 生活指标 -->
          <div class="indicators-section">
            <label class="section-label">生活状态</label>
            <div class="indicators-grid">
              <div class="indicator-item">
                <span class="indicator-label">睡眠质量</span>
                <el-rate v-model="sleepQuality" :max="5" show-score />
              </div>
              <div class="indicator-item">
                <span class="indicator-label">压力水平</span>
                <el-rate v-model="stressLevel" :max="5" show-score />
              </div>
            </div>
          </div>

          <!-- 日记内容 -->
          <div class="diary-content-section">
            <label class="section-label">记录今天的感受</label>
            <el-input
              type="textarea"
              :autosize="{ minRows: 4, maxRows: 8 }"
              v-model="diaryContent"
              placeholder="写下你今天的感受、想法或者发生的事情..."
              class="diary-textarea"
            />
          </div>

          <!-- 提交按钮 -->
          <div class="submit-section">
            <el-button v-if="isEditing" @click="cancelEdit" size="large">
              取消
            </el-button>
            <el-button type="primary" size="large" @click="submitDiary">
              <el-icon><Promotion /></el-icon>
              {{ isEditing ? '更新日记' : '保存日记' }}
            </el-button>
          </div>
        </div>
      </div>

      <!-- 历史记录 -->
      <div class="history-section">
        <div class="section-header">
          <h2>历史记录</h2>
          <span class="record-count" v-if="totalRecords > 0">共 {{ totalRecords }} 条记录</span>
        </div>

        <div class="history-list" v-if="paginatedList.length > 0">
          <div
            v-for="item in paginatedList"
            :key="item.id"
            class="history-item"
          >
            <div class="history-header">
              <div class="history-date">
                <span class="emoji">{{ getMoodEmoji(item.moodScore) }}</span>
                <span class="date">{{ item.date }}</span>
              </div>
              <el-rate :model-value="item.moodScore" disabled show-score :max="10" />
            </div>
            <div class="history-content">{{ item.content }}</div>
            <div class="history-indicators" v-if="item.sleepQuality || item.stressLevel">
              <span class="indicator-tag" v-if="item.sleepQuality">
                <el-icon><Moon /></el-icon>
                睡眠: {{ item.sleepQuality }}/5
              </span>
              <span class="indicator-tag" v-if="item.stressLevel">
                <el-icon><Lightning /></el-icon>
                压力: {{ item.stressLevel }}/5
              </span>
            </div>
            <div class="history-actions">
              <el-button type="primary" text size="small" @click="editDiary(item)">
                <el-icon><Edit /></el-icon>
                编辑
              </el-button>
              <el-button type="danger" text size="small" @click="deleteDiary(item)">
                <el-icon><Delete /></el-icon>
                删除
              </el-button>
            </div>
          </div>
        </div>
        <el-empty v-else description="还没有日记记录" />

        <!-- 分页 -->
        <div class="pagination-section" v-if="totalPages > 1">
          <el-pagination
            v-model:current-page="currentPage"
            :page-size="pageSize"
            :total="totalRecords"
            layout="prev, pager, next"
            @current-change="handlePageChange"
          />
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from "vue";
import { ElMessage, ElMessageBox } from "element-plus";
import like from "../assets/images/like.png";
import { Promotion, Edit, Delete, Moon, Lightning } from "@element-plus/icons-vue";

const MAX_RECORDS = 100;
const pageSize = ref(10);
const currentPage = ref(1);

const displayDate = computed(() => {
  if (isEditing.value && editingDate.value) {
    return formatDateString(editingDate.value);
  }
  const date = new Date();
  return date.toLocaleDateString("zh-CN", {
    year: "numeric",
    month: "long",
    day: "numeric",
    weekday: "long",
  });
});

const editingDate = ref("");

const formatDateString = (dateStr) => {
  try {
    const date = new Date(dateStr);
    return date.toLocaleDateString("zh-CN", {
      year: "numeric",
      month: "long",
      day: "numeric",
      weekday: "long",
    });
  } catch {
    return dateStr;
  }
};

const moodOptions = [
  { value: 1, emoji: "😢", label: "难过" },
  { value: 3, emoji: "😔", label: "低落" },
  { value: 5, emoji: "😐", label: "平静" },
  { value: 7, emoji: "😊", label: "愉快" },
  { value: 10, emoji: "😄", label: "开心" },
];

const currentMood = ref(5);
const moodScore = ref(5);
const sleepQuality = ref(3);
const stressLevel = ref(3);
const diaryContent = ref("");
const historyList = ref([]);
const isEditing = ref(false);
const editingId = ref(null);

const totalRecords = computed(() => historyList.value.length);

const totalPages = computed(() => Math.ceil(totalRecords.value / pageSize.value));

const paginatedList = computed(() => {
  const start = (currentPage.value - 1) * pageSize.value;
  const end = start + pageSize.value;
  return historyList.value.slice(start, end);
});

const selectMood = (value) => {
  currentMood.value = value;
  moodScore.value = value;
};

const getMoodEmoji = (score) => {
  if (score <= 2) return "😢";
  if (score <= 4) return "😔";
  if (score <= 6) return "😐";
  if (score <= 8) return "😊";
  return "😄";
};

const submitDiary = () => {
  if (!diaryContent.value.trim()) {
    ElMessage.warning("请输入日记内容");
    return;
  }

  if (isEditing.value && editingId.value) {
    const index = historyList.value.findIndex((item) => item.id === editingId.value);
    if (index !== -1) {
      historyList.value[index] = {
        ...historyList.value[index],
        moodScore: moodScore.value,
        sleepQuality: sleepQuality.value,
        stressLevel: stressLevel.value,
        content: diaryContent.value,
        updatedAt: new Date().toLocaleString("zh-CN"),
      };
      ElMessage.success("日记更新成功！");
    }
    cancelEdit();
    return;
  }

  if (historyList.value.length >= MAX_RECORDS) {
    ElMessageBox.confirm(
      `日记记录已达上限（${MAX_RECORDS}条），是否删除最早的记录来添加新记录？`,
      "记录上限提醒",
      {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      }
    )
      .then(() => {
        historyList.value.pop();
        addNewDiary();
      })
      .catch(() => {
        ElMessage.info("已取消保存");
      });
    return;
  }

  addNewDiary();
};

const addNewDiary = () => {
  const newDiary = {
    id: Date.now(),
    date: new Date().toLocaleDateString("zh-CN"),
    moodScore: moodScore.value,
    sleepQuality: sleepQuality.value,
    stressLevel: stressLevel.value,
    content: diaryContent.value,
    createdAt: new Date().toLocaleString("zh-CN"),
  };

  historyList.value.unshift(newDiary);
  ElMessage.success("日记保存成功！");

  resetForm();
  currentPage.value = 1;
};

const editDiary = (item) => {
  isEditing.value = true;
  editingId.value = item.id;
  editingDate.value = item.date;
  moodScore.value = item.moodScore;
  currentMood.value = item.moodScore;
  sleepQuality.value = item.sleepQuality || 3;
  stressLevel.value = item.stressLevel || 3;
  diaryContent.value = item.content;

  const formCard = document.getElementById("diary-form-card");
  if (formCard) {
    formCard.scrollIntoView({ behavior: "smooth", block: "start" });
  }
};

const cancelEdit = () => {
  isEditing.value = false;
  editingId.value = null;
  editingDate.value = "";
  resetForm();
};

const resetForm = () => {
  moodScore.value = 5;
  currentMood.value = 5;
  sleepQuality.value = 3;
  stressLevel.value = 3;
  diaryContent.value = "";
};

const deleteDiary = (item) => {
  ElMessageBox.confirm("确定要删除这条日记记录吗？", "删除确认", {
    confirmButtonText: "删除",
    cancelButtonText: "取消",
    type: "warning",
  })
    .then(() => {
      const index = historyList.value.findIndex((d) => d.id === item.id);
      if (index !== -1) {
        historyList.value.splice(index, 1);
        ElMessage.success("删除成功");

        if (paginatedList.value.length === 0 && currentPage.value > 1) {
          currentPage.value--;
        }
      }
    })
    .catch(() => {
      // 用户取消
    });
};

const handlePageChange = (page) => {
  currentPage.value = page;
};

onMounted(() => {
  historyList.value = [
    {
      id: 1,
      date: "2024-01-15",
      moodScore: 7,
      sleepQuality: 4,
      stressLevel: 2,
      content: "今天天气不错，下午去公园散步，心情很愉快。",
    },
    {
      id: 2,
      date: "2024-01-14",
      moodScore: 5,
      sleepQuality: 3,
      stressLevel: 4,
      content: "工作有点忙，但完成了重要的任务，感觉还不错。",
    },
  ];
});
</script>

<style lang="scss" scoped>
.emotional-diary-page {
  min-height: 100vh;
  padding: 40px 20px;
  background: linear-gradient(
    135deg,
    rgba(255, 255, 255, 0.95) 0%,
    rgba(255, 252, 248, 0.98) 100%
  );

  @keyframes breathing {
    0%,
    100% {
      transform: scale(1);
      box-shadow: 0 6px 24px rgba(251, 146, 60, 0.25);
    }
    50% {
      transform: scale(1.1);
      box-shadow: 0 12px 40px rgba(251, 146, 60, 0.4);
    }
  }

  .page-container {
    max-width: 800px;
    margin: 0 auto;
    width: 100%;
  }

  .hero-section {
    text-align: center;
    margin-bottom: 40px;

    .breathing-circle {
      width: 80px;
      height: 80px;
      background: linear-gradient(135deg, #fb923c 0%, #f59e0b 100%);
      border-radius: 50%;
      display: flex;
      align-items: center;
      justify-content: center;
      margin: 0 auto 20px;
      animation: breathing 4s ease-in-out infinite;
      box-shadow: 0 6px 24px rgba(251, 146, 60, 0.25);
    }

    .page-title {
      font-size: 32px;
      font-weight: 700;
      background: linear-gradient(135deg, #fb923c, #f59e0b);
      -webkit-background-clip: text;
      -webkit-text-fill-color: transparent;
      background-clip: text;
      margin: 0 0 12px;
    }

    .page-desc {
      font-size: 16px;
      color: #8b7fc0;
      margin: 0;
    }
  }

  .diary-form-section {
    margin-bottom: 40px;
  }

  .form-card {
    background: linear-gradient(
      135deg,
      rgba(255, 255, 255, 0.95) 0%,
      rgba(255, 252, 250, 0.98) 100%
    );
    border-radius: 20px;
    padding: 32px;
    box-shadow:
      0 12px 40px rgba(251, 146, 60, 0.08),
      0 4px 16px rgba(0, 0, 0, 0.04);
    border: 1px solid rgba(251, 146, 60, 0.1);
    backdrop-filter: blur(10px);

    .card-header {
      display: flex;
      justify-content: space-between;
      align-items: center;
      margin-bottom: 32px;
      padding-bottom: 20px;
      border-bottom: 1px solid rgba(251, 146, 60, 0.1);

      h2 {
        font-size: 24px;
        font-weight: 700;
        color: #333;
        margin: 0;
      }

      .current-date {
        font-size: 14px;
        color: #8b7fc0;
        font-weight: 500;
      }
    }

    .section-label {
      display: block;
      font-size: 16px;
      font-weight: 600;
      color: #333;
      margin-bottom: 16px;
    }

    .mood-section,
    .indicators-section,
    .diary-content-section {
      margin-bottom: 32px;
    }

    .mood-options {
      display: flex;
      justify-content: center;
      gap: 16px;
      margin-bottom: 24px;
      flex-wrap: wrap;

      .mood-item {
        display: flex;
        flex-direction: column;
        align-items: center;
        gap: 8px;
        padding: 16px 24px;
        border-radius: 16px;
        cursor: pointer;
        transition: all 0.3s ease;
        border: 2px solid transparent;
        background: #fafafa;

        &:hover {
          background: #fff7ed;
          border-color: #fed7aa;
          transform: translateY(-2px);
        }

        &.active {
          background: linear-gradient(135deg, #fff7ed, #ffedd5);
          border-color: #fb923c;
          box-shadow: 0 4px 12px rgba(251, 146, 60, 0.2);
        }

        .mood-emoji {
          font-size: 32px;
        }

        .mood-label {
          font-size: 14px;
          color: #666;
          font-weight: 500;
        }
      }
    }

    .mood-rating {
      display: flex;
      justify-content: center;
    }

    .indicators-grid {
      display: grid;
      grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
      gap: 24px;

      .indicator-item {
        display: flex;
        flex-direction: column;
        gap: 12px;
        padding: 20px;
        background: #fafafa;
        border-radius: 12px;
        border: 1px solid rgba(251, 146, 60, 0.1);

        .indicator-label {
          font-size: 14px;
          color: #666;
          font-weight: 500;
        }
      }
    }

    .diary-textarea {
      :deep(textarea) {
        border-radius: 12px;
        border: 1px solid rgba(251, 146, 60, 0.2);
        background: #fff;
        padding: 16px;
        font-size: 15px;
        line-height: 1.6;

        &:focus {
          border-color: #fb923c;
        }
      }
    }

    .submit-section {
      display: flex;
      justify-content: center;
      gap: 16px;

      .el-button {
        &--primary {
          background: linear-gradient(135deg, #fb923c 0%, #f59e0b 100%);
          border: none;
          padding: 0 48px;
          height: 48px;
          font-size: 16px;
          font-weight: 600;
          box-shadow: 0 6px 20px rgba(251, 146, 60, 0.25);
          transition: all 0.3s ease;

          &:hover {
            transform: translateY(-2px);
            box-shadow: 0 8px 24px rgba(251, 146, 60, 0.35);
          }
        }
      }
    }
  }

  .history-section {
    .section-header {
      margin-bottom: 24px;
      display: flex;
      justify-content: space-between;
      align-items: center;

      h2 {
        font-size: 24px;
        font-weight: 700;
        color: #333;
        margin: 0;
      }

      .record-count {
        font-size: 14px;
        color: #8b7fc0;
      }
    }

    .history-list {
      display: flex;
      flex-direction: column;
      gap: 16px;

      .history-item {
        background: white;
        border-radius: 16px;
        padding: 24px;
        box-shadow: 0 4px 12px rgba(0, 0, 0, 0.04);
        border: 1px solid rgba(251, 146, 60, 0.08);
        transition: all 0.3s ease;

        &:hover {
          box-shadow: 0 8px 20px rgba(251, 146, 60, 0.1);
          transform: translateY(-2px);
        }

        .history-header {
          display: flex;
          justify-content: space-between;
          align-items: center;
          margin-bottom: 12px;

          .history-date {
            display: flex;
            align-items: center;
            gap: 8px;

            .emoji {
              font-size: 24px;
            }

            .date {
              font-size: 15px;
              color: #333;
              font-weight: 600;
            }
          }
        }

        .history-content {
          color: #666;
          line-height: 1.6;
          font-size: 15px;
          margin-bottom: 12px;
        }

        .history-indicators {
          display: flex;
          gap: 16px;
          margin-bottom: 12px;

          .indicator-tag {
            display: flex;
            align-items: center;
            gap: 4px;
            font-size: 13px;
            color: #8b7fc0;
            background: #f8f9ff;
            padding: 4px 12px;
            border-radius: 12px;
          }
        }

        .history-actions {
          display: flex;
          justify-content: flex-end;
          gap: 8px;
          padding-top: 12px;
          border-top: 1px solid rgba(251, 146, 60, 0.08);

          .el-button {
            font-weight: 500;
          }
        }
      }
    }

    .pagination-section {
      display: flex;
      justify-content: center;
      margin-top: 24px;

      :deep(.el-pager li.is-active) {
        background: linear-gradient(135deg, #fb923c 0%, #f59e0b 100%);
      }
    }
  }
}

@media (max-width: 768px) {
  .emotional-diary-page {
    padding: 20px 16px;

    .page-container {
      width: 100%;
    }

    .hero-section {
      .page-title {
        font-size: 24px;
      }

      .page-desc {
        font-size: 14px;
      }
    }

    .form-card {
      padding: 20px;

      .card-header {
        flex-direction: column;
        gap: 8px;
        align-items: flex-start;
      }

      .mood-options {
        gap: 8px;

        .mood-item {
          padding: 12px 16px;

          .mood-emoji {
            font-size: 24px;
          }

          .mood-label {
            font-size: 12px;
          }
        }
      }
    }

    .history-section {
      .history-item {
        .history-actions {
          flex-wrap: wrap;
        }
      }
    }
  }
}
</style>
