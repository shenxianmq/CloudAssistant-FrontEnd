<template>
    <!-- 只有当syncConfig.id不为空时再呈现页面 -->
    <div>
        <v-tabs v-model="activeTab" show-arrows class="v-tabs-pill">
            <v-tab v-for="item in tabs" :key="item.icon" :value="item.tab"
                selected-class="v-slide-group-item--active v-tab--selected" @click="jumpTab(item.tab)">
                <div>
                    <v-icon size="20" start :icon="item.icon" />
                    {{ item.title }}
                </div>
            </v-tab>
        </v-tabs>
        <v-window v-model="activeTab" class="mt-5 disable-tab-transition" :touch="false">
            <!-- 用户 -->
            <v-window-item value="symlink">
                <transition name="fade-slide" appear>
                    <div>
                        <SymlinkSettings v-model:syncConfig="syncConfig" v-model:syncTemplate="syncTemplate"
                            :isNew="true">
                            <template #footer>
                                <div class="btn-settings">
                                    <v-btn @click="saveConfig">保存</v-btn>
                                </div>
                            </template>
                        </SymlinkSettings>
                    </div>
                </transition>
            </v-window-item>
            <v-window-item value="scheduled_task">
                <transition name="fade-slide" appear>
                    <div>
                        <ScheduledSettings v-model:syncConfig="syncConfig">
                            <template #footer>
                                <div class="btn-settings">
                                    <v-btn @click="saveConfig">保存</v-btn>
                                </div>
                            </template>
                        </ScheduledSettings>
                    </div>
                </transition>
            </v-window-item>
            <v-window-item value="sync_observer">
                <transition name="fade-slide" appear>
                    <div>
                        <ObserverSettings v-model:syncConfig="syncConfig">
                            <template #footer>
                                <div class="btn-settings">
                                    <v-btn @click="saveConfig">保存</v-btn>
                                </div>
                            </template>
                        </ObserverSettings>
                    </div>
                </transition>
            </v-window-item>
            <v-window-item value="cloud_status">
                <transition name="fade-slide" appear>
                    <div>
                        <CloudStatus v-model:syncConfig="syncConfig">
                            <template #footer>
                                <div class="btn-settings">
                                    <v-btn @click="saveConfig">保存</v-btn>
                                </div>
                            </template>
                        </CloudStatus>
                    </div>
                </transition>
            </v-window-item>
            <v-window-item value="tree_sync">
                <transition name="fade-slide" appear>
                    <div>
                        <TreeSync v-model:syncConfig="syncConfig">
                            <template #footer>
                                <div class="btn-settings">
                                    <v-btn @click="saveConfig">保存</v-btn>
                                </div>
                            </template>
                        </TreeSync>
                    </div>
                </transition>
            </v-window-item>
        </v-window>
    </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import router from '@/router'
import { useRoute } from 'vue-router'
import api from '@/api/index'
import { SyncItem, SaveResponse } from '@/api/types';
import { SyncItemVar, SyncTemplateVar } from '@/api/variable'
import SymlinkSettings from '@/views/sync_config/SymlinkSettings.vue'
import ScheduledSettings from '@/views/sync_config/ScheduledSettings.vue'
import ObserverSettings from '@/views/sync_config/ObserverSettings.vue'
import CloudStatus from '@/views/sync_config/CloudStatus.vue'
import TreeSync from '@/views/sync_config/TreeSync.vue'
import { useToast } from 'vue-toast-notification';
const $toast = useToast()
const route = useRoute()

const syncConfig = ref(<SyncItem>{ ...SyncItemVar })

const activeTab = ref(route.query.tab)

const tabs = [{ tab: "symlink", icon: "mdi-link-variant", title: "软链接" }, { tab: "scheduled_task", icon: "mdi-calendar-clock", title: "定时任务" }, { tab: "sync_observer", icon: "mdi-eye", title: "实时监控" }, { tab: "cloud_status", icon: "mdi-weather-cloudy", title: "掉盘检测" }, { tab: "tree_sync", icon: "mdi-sync", title: "目录树同步" }]

const syncTemplate = ref(<SyncItem[]>[{ ...SyncTemplateVar }])

async function fetchSyncTemplate() {
    try {
        const data: SyncItem[] = await api.get(`/autosymlink/sync_list`)
        if (data.length > 0) {
            syncTemplate.value = data
        }
    } catch (error) {
        console.error('Error fetching sync config:', error)
    }
    try {
        const data: SyncItem = await api.get(`/autosymlink/sync_template`)
        syncConfig.value = data

    } catch (error) {
        console.error('Error fetching sync config:', error)
    }
}

async function saveConfig() {
    try {
        const response: SaveResponse = await api.post(`/autosymlink/add_sync/save_config`, syncConfig.value)
        if (response.success) {
            $toast.success(response.message)
        }
        else (
            $toast.error(response.message)
        )
        setTimeout(() => {
            router.push('/sync_list')
        }, 1000); // 2 秒后执行 greet 函数
    } catch (error) {
        console.error('Error fetching sync config:', error)
    }
}

function jumpTab(tab: string) {
    router.push('/add_sync?tab=' + tab)
}

onMounted(() => {
    fetchSyncTemplate();
});
</script>

<style scoped>
.v-tab-item--selected {
    background: rgba(var(--v-theme-primary));
    color: white !important;
}

:deep(.v-tab__slider) {
    display: none !important;
}

/* .v-window-item {
    transition: none !important;
} */
</style>