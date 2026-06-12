<template>
  <section class="manage-page">
    <div class="page-head">
      <div>
        <h1>回收站</h1>
        <p>误删的展览、展品、导览和批注可在此恢复或彻底删除，清空前会按模块显示待删条目数。</p>
      </div>
      <n-button type="error" :disabled="totalDeleted === 0" @click="handleClearAll">清空回收站</n-button>
    </div>

    <n-tabs v-model:value="activeTab" type="line" animated>
      <n-tab-pane name="exhibition" :tab="tabLabel('展览', exhibitionStore.deletedExhibitions.length)">
        <div class="trash-list">
          <n-empty v-if="exhibitionStore.deletedExhibitions.length === 0" description="暂无已删除的展览" />
          <div v-for="item in exhibitionStore.deletedExhibitions" :key="item.id" class="trash-item">
            <div class="trash-item-info">
              <strong>{{ item.title }}</strong>
              <span class="trash-meta">{{ item.curator }} · 删除于 {{ formatDate(item.deletedAt) }}</span>
            </div>
            <div class="trash-item-actions">
              <n-button size="small" secondary @click="restoreExhibition(item.id)">恢复</n-button>
              <n-button size="small" quaternary type="error" @click="permanentDeleteExhibition(item.id)">彻底删除</n-button>
            </div>
          </div>
        </div>
      </n-tab-pane>

      <n-tab-pane name="artifact" :tab="tabLabel('展品', artifactStore.deletedArtifacts.length)">
        <div class="trash-list">
          <n-empty v-if="artifactStore.deletedArtifacts.length === 0" description="暂无已删除的展品" />
          <div v-for="item in artifactStore.deletedArtifacts" :key="item.id" class="trash-item">
            <div class="trash-item-info">
              <img v-if="item.images[0]" :src="item.images[0]" class="trash-thumb" alt="" />
              <div>
                <strong>{{ item.name }}</strong>
                <span class="trash-meta">{{ item.author }} · 删除于 {{ formatDate(item.deletedAt) }}</span>
              </div>
            </div>
            <div class="trash-item-actions">
              <n-button size="small" secondary @click="restoreArtifact(item.id)">恢复</n-button>
              <n-button size="small" quaternary type="error" @click="permanentDeleteArtifact(item.id)">彻底删除</n-button>
            </div>
          </div>
        </div>
      </n-tab-pane>

      <n-tab-pane name="tour" :tab="tabLabel('导览', tourStore.deletedTours.length)">
        <div class="trash-list">
          <n-empty v-if="tourStore.deletedTours.length === 0" description="暂无已删除的导览" />
          <div v-for="item in tourStore.deletedTours" :key="item.id" class="trash-item">
            <div class="trash-item-info">
              <strong>{{ item.name }}</strong>
              <span class="trash-meta">{{ item.nodes.length }} 个节点 · 删除于 {{ formatDate(item.deletedAt) }}</span>
            </div>
            <div class="trash-item-actions">
              <n-button size="small" secondary @click="restoreTour(item.id)">恢复</n-button>
              <n-button size="small" quaternary type="error" @click="permanentDeleteTour(item.id)">彻底删除</n-button>
            </div>
          </div>
        </div>
      </n-tab-pane>

      <n-tab-pane name="annotation" :tab="tabLabel('批注', annotationStore.deletedAnnotations.length)">
        <div class="trash-list">
          <n-empty v-if="annotationStore.deletedAnnotations.length === 0" description="暂无已删除的批注" />
          <div v-for="item in annotationStore.deletedAnnotations" :key="item.id" class="trash-item">
            <div class="trash-item-info">
              <strong>{{ item.title }}</strong>
              <span class="trash-meta">{{ item.content.slice(0, 30) }} · 删除于 {{ formatDate(item.deletedAt) }}</span>
            </div>
            <div class="trash-item-actions">
              <n-button size="small" secondary @click="restoreAnnotation(item.id)">恢复</n-button>
              <n-button size="small" quaternary type="error" @click="permanentDeleteAnnotation(item.id)">彻底删除</n-button>
            </div>
          </div>
        </div>
      </n-tab-pane>
    </n-tabs>

    <n-modal v-model:show="clearAllVisible" preset="dialog" type="warning" title="确认清空回收站" positive-text="确认清空" negative-text="取消" @positive-click="doClearAll">
      <p>以下内容将被彻底删除，操作不可撤销：</p>
      <ul class="clear-summary">
        <li v-if="exhibitionStore.deletedExhibitions.length">展览 × {{ exhibitionStore.deletedExhibitions.length }}</li>
        <li v-if="artifactStore.deletedArtifacts.length">展品 × {{ artifactStore.deletedArtifacts.length }}</li>
        <li v-if="tourStore.deletedTours.length">导览 × {{ tourStore.deletedTours.length }}</li>
        <li v-if="annotationStore.deletedAnnotations.length">批注 × {{ annotationStore.deletedAnnotations.length }}</li>
      </ul>
    </n-modal>
  </section>
</template>

<script setup lang="ts">
import { computed, ref } from 'vue';
import { useMessage, useDialog } from 'naive-ui';
import { useAnnotationStore } from '@/stores/annotation';
import { useArtifactStore } from '@/stores/artifact';
import { useExhibitionStore } from '@/stores/exhibition';
import { useTourStore } from '@/stores/tour';

const message = useMessage();
const dialog = useDialog();
const exhibitionStore = useExhibitionStore();
const artifactStore = useArtifactStore();
const tourStore = useTourStore();
const annotationStore = useAnnotationStore();

const activeTab = ref('exhibition');
const clearAllVisible = ref(false);

const totalDeleted = computed(
  () =>
    exhibitionStore.deletedExhibitions.length +
    artifactStore.deletedArtifacts.length +
    tourStore.deletedTours.length +
    annotationStore.deletedAnnotations.length
);

function tabLabel(label: string, count: number): string {
  return count > 0 ? `${label} (${count})` : label;
}

function formatDate(iso?: string): string {
  if (!iso) return '—';
  const d = new Date(iso);
  return `${d.getFullYear()}-${String(d.getMonth() + 1).padStart(2, '0')}-${String(d.getDate()).padStart(2, '0')} ${String(d.getHours()).padStart(2, '0')}:${String(d.getMinutes()).padStart(2, '0')}`;
}

async function restoreExhibition(id: string) {
  await exhibitionStore.restoreExhibition(id);
  message.success('展览已恢复');
}

async function permanentDeleteExhibition(id: string) {
  dialog.warning({
    title: '彻底删除',
    content: '该展览将被永久删除，无法恢复。确认？',
    positiveText: '确认删除',
    negativeText: '取消',
    onPositiveClick: async () => {
      await exhibitionStore.permanentDeleteExhibition(id);
      message.success('展览已彻底删除');
    }
  });
}

async function restoreArtifact(id: string) {
  await artifactStore.restoreArtifact(id);
  message.success('展品已恢复');
}

async function permanentDeleteArtifact(id: string) {
  dialog.warning({
    title: '彻底删除',
    content: '该展品及其关联文件将被永久删除，无法恢复。确认？',
    positiveText: '确认删除',
    negativeText: '取消',
    onPositiveClick: async () => {
      await artifactStore.permanentDeleteArtifact(id);
      message.success('展品已彻底删除');
    }
  });
}

async function restoreTour(id: string) {
  await tourStore.restoreTour(id);
  message.success('导览已恢复');
}

async function permanentDeleteTour(id: string) {
  dialog.warning({
    title: '彻底删除',
    content: '该导览将被永久删除，无法恢复。确认？',
    positiveText: '确认删除',
    negativeText: '取消',
    onPositiveClick: async () => {
      await tourStore.permanentDeleteTour(id);
      message.success('导览已彻底删除');
    }
  });
}

async function restoreAnnotation(id: string) {
  await annotationStore.restoreAnnotation(id);
  message.success('批注已恢复');
}

async function permanentDeleteAnnotation(id: string) {
  dialog.warning({
    title: '彻底删除',
    content: '该批注将被永久删除，无法恢复。确认？',
    positiveText: '确认删除',
    negativeText: '取消',
    onPositiveClick: async () => {
      await annotationStore.permanentDeleteAnnotation(id);
      message.success('批注已彻底删除');
    }
  });
}

function handleClearAll() {
  clearAllVisible.value = true;
}

async function doClearAll() {
  await Promise.all([
    ...exhibitionStore.deletedExhibitions.map((e) => exhibitionStore.permanentDeleteExhibition(e.id)),
    ...artifactStore.deletedArtifacts.map((a) => artifactStore.permanentDeleteArtifact(a.id)),
    ...tourStore.deletedTours.map((t) => tourStore.permanentDeleteTour(t.id)),
    ...annotationStore.deletedAnnotations.map((a) => annotationStore.permanentDeleteAnnotation(a.id))
  ]);
  message.success('回收站已清空');
}
</script>

<style scoped>
.manage-page {
  display: grid;
  gap: 18px;
}

.trash-list {
  display: grid;
  gap: 10px;
  padding-top: 8px;
}

.trash-item {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 16px;
  padding: 14px 18px;
  background: rgba(251, 245, 232, 0.82);
  border: 1px solid rgba(23, 63, 53, 0.14);
  border-radius: 8px;
}

.trash-item-info {
  display: flex;
  align-items: center;
  gap: 12px;
  min-width: 0;
}

.trash-item-info strong {
  font-size: 15px;
}

.trash-thumb {
  width: 40px;
  height: 40px;
  object-fit: cover;
  border-radius: 4px;
  flex-shrink: 0;
}

.trash-item-info > div {
  display: grid;
  gap: 2px;
  min-width: 0;
}

.trash-meta {
  color: rgba(31, 46, 41, 0.6);
  font-size: 13px;
}

.trash-item-actions {
  display: flex;
  flex-shrink: 0;
  gap: 8px;
}

.clear-summary {
  margin: 8px 0 0;
  padding-left: 20px;
  line-height: 1.9;
}

@media (max-width: 640px) {
  .trash-item {
    flex-direction: column;
    align-items: flex-start;
  }
}
</style>
