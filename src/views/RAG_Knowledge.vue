<template>
  <div class="knowledge-page">
    <div class="page-container">
      <div class="hero-section">
        <div class="breathing-circle">
          <el-image :src="robotFill" fit="cover" style="width: 40px; height: 40px"></el-image>
        </div>
        <h1 class="page-title">知识库</h1>
        <p class="page-desc">探索心理健康知识与实用技巧</p>
      </div>

      <!-- 分类筛选 -->
      <div class="filter-section" v-if="categoryList.length > 0">
        <el-scrollbar class="category-scrollbar">
          <div class="category-list">
            <div
              class="category-item"
              :class="{ active: currentCategory === null }"
              @click="selectCategory(null)"
            >
              全部
            </div>
            <div
              class="category-item"
              :class="{ active: currentCategory === item.id }"
              v-for="item in categoryList"
              :key="item.id"
              @click="selectCategory(item.id)"
            >
              {{ item.label }}
            </div>
          </div>
        </el-scrollbar>
      </div>

      <!-- 搜索框 -->
      <div class="search-section">
        <el-input
          v-model="searchKeyword"
          placeholder="搜索感兴趣的内容..."
          size="large"
          clearable
          class="search-input"
          @keyup.enter="handleSearch"
        >
          <template #prefix>
            <el-icon><Search /></el-icon>
          </template>
        </el-input>
      </div>

      <!-- 文章列表 -->
      <div class="articles-section">
        <div class="section-header">
          <h2>{{ currentCategory ? '精选文章' : '全部文章' }}</h2>
          <span class="article-count" v-if="filteredTotal > 0">共 {{ filteredTotal }} 篇</span>
        </div>

        <div v-if="loading" class="loading-container">
          <el-icon class="is-loading"><Loading /></el-icon>
        </div>

        <div v-else-if="currentPageArticles.length > 0" class="articles-grid">
          <div
            v-for="article in currentPageArticles"
            :key="article.id"
            class="article-card"
            @click="viewArticle(article)"
          >
            <div class="article-cover" v-if="article.cover">
              <el-image :src="article.cover" fit="cover" />
            </div>
            <div class="article-content">
              <div class="article-meta">
                <span class="article-category" v-if="article.categoryName">
                  {{ article.categoryName }}
                </span>
                <span class="article-date">
                  {{ article.updatedAt || article.createdAt }}
                </span>
              </div>
              <h3 class="article-title">{{ article.title }}</h3>
              <p class="article-summary" v-if="article.summary">
                {{ article.summary }}
              </p>
              <div class="article-footer">
                <div class="article-stats">
                  <span class="stat-item">
                    <el-icon><View /></el-icon>
                    {{ article.readCount || article.views || 0 }}
                  </span>
                </div>
                <el-button type="primary" text size="small">
                  阅读更多
                </el-button>
              </div>
            </div>
          </div>
        </div>

        <el-empty v-else description="暂无文章" />
      </div>

      <!-- 分页 -->
      <div class="pagination-section" v-if="filteredTotal > pageSize">
        <el-pagination
          v-model:current-page="currentPage"
          :page-size="pageSize"
          :total="filteredTotal"
          layout="prev, pager, next"
        />
      </div>
    </div>

    <!-- 文章详情弹窗 -->
    <el-dialog
      v-model="articleDialogVisible"
      :title="currentArticle?.title"
      width="800px"
      class="article-dialog"
      destroy-on-close
    >
      <div class="article-detail" v-if="currentArticle">
        <div class="detail-meta">
          <span class="detail-category" v-if="currentArticle.categoryName">
            {{ currentArticle.categoryName }}
          </span>
          <span class="detail-date">
            {{ currentArticle.updatedAt || currentArticle.createdAt }}
          </span>
          <span class="detail-views">
            <el-icon><View /></el-icon>
            {{ currentArticle.readCount || currentArticle.views || 0 }} 阅读
          </span>
        </div>
        <div class="detail-content" v-if="currentArticle.content">
          <div v-html="currentArticle.content"></div>
        </div>
        <div v-else class="detail-content">
          <p>{{ currentArticle.summary || '文章内容正在加载...' }}</p>
        </div>
      </div>
    </el-dialog>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from "vue";
import { ElMessage } from "element-plus";
import { Search, View, Loading } from "@element-plus/icons-vue";
import robotFill from "../assets/images/robot-fill.png";
import { categoryTree, articlePage } from "../api/admin";

const searchKeyword = ref("");
const currentCategory = ref(null);
const categoryList = ref([]);
const articles = ref([]);
const loading = ref(false);
const currentPage = ref(1);
const pageSize = ref(12);
const total = ref(0);
const articleDialogVisible = ref(false);
const currentArticle = ref(null);

const filteredArticles = computed(() => {
  if (!searchKeyword.value) {
    return articles.value;
  }
  const keyword = searchKeyword.value.toLowerCase();
  return articles.value.filter(
    (article) =>
      article.title?.toLowerCase().includes(keyword) ||
      article.summary?.toLowerCase().includes(keyword)
  );
});

const filteredTotal = computed(() => filteredArticles.value.length);

const currentPageArticles = computed(() => {
  const start = (currentPage.value - 1) * pageSize.value;
  const end = start + pageSize.value;
  return filteredArticles.value.slice(start, end);
});

const selectCategory = (categoryId) => {
  currentCategory.value = categoryId;
  currentPage.value = 1;
  loadArticles();
};

const handleSearch = () => {
  currentPage.value = 1;
};

const loadCategories = async () => {
  try {
    const res = await categoryTree();
    const data = res?.data || res;
    if (Array.isArray(data)) {
      categoryList.value = formatCategoryList(data);
    }
  } catch (error) {
    console.error("加载分类失败:", error);
  }
};

const formatCategoryList = (nodes, result = []) => {
  nodes.forEach((node) => {
    result.push({
      id: node.id,
      label: node.categoryName,
    });
    if (node.children && node.children.length > 0) {
      formatCategoryList(node.children, result);
    }
  });
  return result;
};

const loadArticles = async () => {
  loading.value = true;
  try {
    const params = {
      currentPage: 1,
      size: 1000,
      pageNum: 1,
      pageSize: 1000,
    };
    if (currentCategory.value) {
      params.categoryId = currentCategory.value;
    }

    const res = await articlePage(params);
    const realData = res?.data?.records ? res.data : res?.data || res;
    const records = realData.records || realData.list || [];
    articles.value = records.filter((item) => item.status === 1);
    currentPage.value = 1;
  } catch (error) {
    ElMessage.error("加载文章失败");
  } finally {
    loading.value = false;
  }
};

const viewArticle = (article) => {
  currentArticle.value = article;
  articleDialogVisible.value = true;
};

onMounted(() => {
  loadCategories();
  loadArticles();
});
</script>

<style lang="scss" scoped>
.knowledge-page {
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
    max-width: 1200px;
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

  .filter-section {
    margin-bottom: 24px;

    .category-scrollbar {
      width: 100%;
    }

    .category-list {
      display: flex;
      gap: 12px;
      padding: 4px;

      .category-item {
        padding: 8px 24px;
        border-radius: 24px;
        background: #fafafa;
        color: #666;
        font-size: 14px;
        font-weight: 500;
        cursor: pointer;
        transition: all 0.3s ease;
        border: 1px solid transparent;
        white-space: nowrap;

        &:hover {
          background: #fff7ed;
          color: #fb923c;
        }

        &.active {
          background: linear-gradient(135deg, #fb923c 0%, #f59e0b 100%);
          color: white;
          border: none;
          box-shadow: 0 4px 12px rgba(251, 146, 60, 0.3);
        }
      }
    }
  }

  .search-section {
    margin-bottom: 32px;

    .search-input {
      :deep(.el-input__wrapper) {
        border-radius: 24px;
        padding: 8px 20px;
        background: white;
        border: 1px solid rgba(251, 146, 60, 0.15);
        box-shadow: 0 4px 12px rgba(251, 146, 60, 0.06);
        transition: all 0.3s ease;

        &:hover {
          border-color: rgba(251, 146, 60, 0.3);
          box-shadow: 0 6px 16px rgba(251, 146, 60, 0.1);
        }

        &.is-focus {
          border-color: #fb923c;
          box-shadow: 0 6px 20px rgba(251, 146, 60, 0.15);
        }
      }
    }
  }

  .articles-section {
    margin-bottom: 32px;

    .section-header {
      display: flex;
      justify-content: space-between;
      align-items: center;
      margin-bottom: 24px;

      h2 {
        font-size: 24px;
        font-weight: 700;
        color: #333;
        margin: 0;
      }

      .article-count {
        font-size: 14px;
        color: #8b7fc0;
      }
    }

    .loading-container {
      display: flex;
      justify-content: center;
      padding: 60px 0;

      .el-icon {
        font-size: 32px;
        color: #fb923c;
      }
    }

    .articles-grid {
      display: grid;
      grid-template-columns: repeat(auto-fill, minmax(320px, 1fr));
      gap: 24px;

      .article-card {
        background: white;
        border-radius: 20px;
        overflow: hidden;
        box-shadow: 0 4px 12px rgba(0, 0, 0, 0.04);
        border: 1px solid rgba(251, 146, 60, 0.08);
        cursor: pointer;
        transition: all 0.3s ease;
        display: flex;
        flex-direction: column;

        &:hover {
          transform: translateY(-4px);
          box-shadow: 0 12px 24px rgba(251, 146, 60, 0.12);
        }

        .article-cover {
          width: 100%;
          height: 180px;
          overflow: hidden;
          background: linear-gradient(135deg, #fff7ed, #ffedd5);

          :deep(.el-image) {
            width: 100%;
            height: 100%;
          }
        }

        .article-content {
          padding: 20px;
          display: flex;
          flex-direction: column;
          flex: 1;

          .article-meta {
            display: flex;
            gap: 12px;
            margin-bottom: 12px;

            .article-category {
              background: linear-gradient(135deg, #fff7ed, #ffedd5);
              color: #fb923c;
              padding: 4px 12px;
              border-radius: 12px;
              font-size: 12px;
              font-weight: 500;
            }

            .article-date {
              color: #999;
              font-size: 12px;
              display: flex;
              align-items: center;
            }
          }

          .article-title {
            font-size: 18px;
            font-weight: 600;
            color: #333;
            margin: 0 0 12px;
            line-height: 1.4;
            display: -webkit-box;
            -webkit-line-clamp: 2;
            -webkit-box-orient: vertical;
            overflow: hidden;
          }

          .article-summary {
            font-size: 14px;
            color: #666;
            line-height: 1.6;
            margin: 0 0 16px;
            display: -webkit-box;
            -webkit-line-clamp: 2;
            -webkit-box-orient: vertical;
            overflow: hidden;
            flex: 1;
          }

          .article-footer {
            display: flex;
            justify-content: space-between;
            align-items: center;
            padding-top: 16px;
            border-top: 1px solid rgba(251, 146, 60, 0.08);

            .article-stats {
              display: flex;
              gap: 16px;

              .stat-item {
                display: flex;
                align-items: center;
                gap: 4px;
                font-size: 13px;
                color: #999;
              }
            }

            .el-button {
              color: #fb923c;
              font-weight: 500;

              &:hover {
                color: #f59e0b;
                background: #fff7ed;
              }
            }
          }
        }
      }
    }
  }

  .pagination-section {
    display: flex;
    justify-content: center;

    :deep(.el-pager li.is-active) {
      background: linear-gradient(135deg, #fb923c 0%, #f59e0b 100%);
    }
  }
}

:deep(.article-dialog) {
  .el-dialog__header {
    background: linear-gradient(135deg, #fb923c 0%, #f59e0b 100%);
    color: white;
    padding: 20px 24px;
    margin: 0;

    .el-dialog__title {
      color: white;
      font-size: 18px;
      font-weight: 600;
    }

    .el-dialog__headerbtn {
      .el-dialog__close {
        color: white;
        font-size: 20px;

        &:hover {
          color: rgba(255, 255, 255, 0.8);
        }
      }
    }
  }

  .el-dialog__body {
    padding: 32px 24px;
  }
}

.article-detail {
  .detail-meta {
    display: flex;
    gap: 16px;
    margin-bottom: 24px;
    padding-bottom: 16px;
    border-bottom: 1px solid rgba(251, 146, 60, 0.1);

    .detail-category {
      background: linear-gradient(135deg, #fff7ed, #ffedd5);
      color: #fb923c;
      padding: 4px 12px;
      border-radius: 12px;
      font-size: 13px;
      font-weight: 500;
    }

    .detail-date,
    .detail-views {
      color: #999;
      font-size: 13px;
      display: flex;
      align-items: center;
      gap: 4px;
    }
  }

  .detail-content {
    font-size: 15px;
    line-height: 1.8;
    color: #333;

    :deep(p) {
      margin-bottom: 16px;
    }

    :deep(h1),
    :deep(h2),
    :deep(h3) {
      margin: 24px 0 12px;
      color: #333;
      font-weight: 600;
    }

    :deep(img) {
      max-width: 100%;
      border-radius: 12px;
      margin: 16px 0;
    }
  }
}

@media (max-width: 768px) {
  .knowledge-page {
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

    .articles-section {
      .articles-grid {
        grid-template-columns: 1fr;
      }
    }
  }
}
</style>
