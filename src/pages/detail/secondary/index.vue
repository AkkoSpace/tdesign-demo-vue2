<template>
  <div>
    <div class="secondary-notification">
      <t-tabs v-model="tabValue">
        <t-tab-panel v-for="(tab, tabIndex) in TAB_LIST" :key="tabIndex" :label="tab.label" :value="tab.value">
          <t-list v-if="msgDataList.length > 0" :split="true" class="secondary-msg-list">
            <t-list-item v-for="(item, index) in msgDataList" :key="index">
              <p :class="['content', { unread: item.status }]" @click="setReadStatus(item)">
                <t-tag :theme="NOTIFICATION_TYPES[item.quality]" size="small" variant="light">
                  {{ item.type }}
                </t-tag>
                {{ item.content }}
              </p>
              <template #action>
                <span class="msg-date">{{ item.date }}</span>
                <div class="msg-action">
                  <t-tooltip
                    :content="item.status ? '设为已读' : '设为未读'"
                    :overlay-style="{ margin: '6px' }"
                    class="set-read-icon"
                  >
                    <span class="msg-action-icon" @click="setReadStatus(item)">
                      <queue-icon v-if="!!item.status" size="16px" />
                      <chat-icon v-else />
                    </span>
                  </t-tooltip>
                  <t-tooltip :overlay-style="{ margin: '6px' }" content="删除通知">
                    <span @click="handleClickDeleteBtn(item)">
                      <delete-icon size="16px" />
                    </span>
                  </t-tooltip>
                </div>
              </template>
            </t-list-item>
          </t-list>
          <div v-else class="secondary-msg-list__empty-list">
            <img alt="空" src="https://tdesign.gtimg.com/pro-template/personal/nothing.png" />
            <p>暂无通知</p>
          </div>
        </t-tab-panel>
      </t-tabs>
    </div>
    <t-dialog
      :body="`确认删除通知：${selectedItem && selectedItem.content}吗？`"
      :onConfirm="deleteMsg"
      :visible.sync="visible"
      header="删除通知"
    ></t-dialog>
  </div>
</template>
<script lang="ts">
import { mapGetters, mapState } from 'vuex';
import { ChatIcon, DeleteIcon, QueueIcon } from 'tdesign-icons-vue';
import { prefix } from '@/config/global';
import { NOTIFICATION_TYPES } from '@/constants';
import { msgDataItem } from '@/store/modules/notification';

const TAB_LIST = [
  {
    label: '全部通知',
    value: 'msgData',
  },
  {
    label: '未读通知',
    value: 'unreadMsg',
  },
  {
    label: '已读通知',
    value: 'readMsg',
  },
];

export default {
  name: 'DetailSecondary',
  components: {
    QueueIcon,
    DeleteIcon,
    ChatIcon,
  },
  data() {
    return {
      NOTIFICATION_TYPES,
      TAB_LIST,
      prefix,
      data: [],
      tabValue: 'msgData',
      visible: false,
      selectedItem: undefined,
    };
  },
  computed: {
    ...mapState('notification', ['msgData']),
    ...mapGetters('notification', ['unreadMsg', 'readMsg']),
    msgDataList() {
      if (this.tabValue === 'msgData') return this.msgData;
      if (this.tabValue === 'unreadMsg') return this.unreadMsg;
      if (this.tabValue === 'readMsg') return this.readMsg;
      return [];
    },
  },
  methods: {
    handleClickDeleteBtn(item: msgDataItem): void {
      this.visible = true;
      this.selectedItem = item;
    },
    setReadStatus(item: msgDataItem): void {
      const changeMsg = this.msgData;
      changeMsg.forEach((e) => {
        if (e.id === item.id) {
          if (e.status) e.status = false;
        }
      });
      this.$store.commit('notification/setMsgData', changeMsg);
    },
    deleteMsg(): void {
      const item = this.selectedItem;
      const changeMsg = this.msgData;
      changeMsg.forEach((e, index) => {
        if (e.id === item.id) {
          changeMsg.splice(index, 1);
        }
      });
      this.visible = false;
      this.$store.commit('notification/setMsgData', changeMsg);
    },
  },
};
</script>
<style lang="less" scoped>
@import './index';
</style>
