<template>
  <t-dialog v-model:visible="formVisible" header="数据" placement="center" :footer="false" >
    <template #body>
      <div class="data-dialog-container dialog-container-padding">
        <div class="data-item">
          <span class="separator"></span>
          <p class="title">配置文件</p>
          <p class="content">数据保存在数据库中, 为方便迁移可导出为json文件, 导入将覆盖原数据</p>
          <div class="config"> 
            <t-collapse expand-mutex>
              <t-collapse-panel value="easyConfig" header="一键配置">
                <t-radio-group v-model="formData.easyConfig.type">
                  <t-radio :value="0">此软件</t-radio>
                  <t-radio :value="1">drpy</t-radio>
                  <t-radio :value="2">tvbox</t-radio>
                  <t-radio :value="3">hipy</t-radio>
                </t-radio-group>
                <p v-if="formData.easyConfig.type === 0" class="tip">请严格遵守本软件接口格式</p>
                <p v-else-if="formData.easyConfig.type === 1" class="tip">目前仅支持sites中type:1的数据,请将js模式设置为0</p>
                <p v-else-if="formData.easyConfig.type === 2" class="tip">目前仅支持sites中type:0或1且的cms类型的数据</p>
                <t-input label="地址：" v-model="formData.easyConfig.url" class="input-item"></t-input>
                <t-popconfirm content="原有数据将清除，确认配置吗" placement="bottom" @confirm="easyConfig">
                  <t-button size="small" block ghost>配置</t-button>
                </t-popconfirm>
              </t-collapse-panel>
              <t-collapse-panel value="remoteImport" header="配置导入">
                <t-radio-group v-model="formData.importData.type">
                  <t-radio value="remote">远端导入</t-radio>
                  <t-radio value="local">本地导入</t-radio>
                </t-radio-group>
                <div v-if="formData.importData.type === 'remote'">
                  <t-input label="地址：" v-model="formData.importData.remoteImpoUrl" class="input-item"></t-input>
                </div>
                <div v-else>
                  <t-upload 
                    v-model="formData.importData.localImpoFile"
                    class="input-item"
                    theme="file"
                    :max="1"
                    accept="application/json"
                    :draggable="true"
                    :request-method="requestMethod"
                  />
                </div>
                <t-popconfirm content="原有数据将清除，确认导入吗" placement="bottom" @confirm="importData">
                  <t-button size="small" block ghost>导入</t-button>
                </t-popconfirm>
              </t-collapse-panel>
              <t-collapse-panel value="setExport" header="配置导出">
                <t-radio v-model="formData.exportSeletct.sites" allow-uncheck class="radio-item">影视源</t-radio>
                <t-radio v-model="formData.exportSeletct.iptv" allow-uncheck class="radio-item">电视源</t-radio>
                <t-radio v-model="formData.exportSeletct.analyze" allow-uncheck class="radio-item">解析源</t-radio>
                <t-radio v-model="formData.exportSeletct.drive" allow-uncheck class="radio-item">网盘源</t-radio>
                <t-radio v-model="formData.exportSeletct.history" allow-uncheck class="radio-item">历史</t-radio>
                <t-radio v-model="formData.exportSeletct.star" allow-uncheck class="radio-item">收藏</t-radio>
                <t-radio v-model="formData.exportSeletct.setting" allow-uncheck class="radio-item">基础配置</t-radio>
                <t-button size="small" block ghost @click="setExport">导出</t-button>
              </t-collapse-panel>
              <t-collapse-panel value="clearData" header="清理数据">
                <t-radio v-model="formData.clearSeletct.sites" allow-uncheck class="radio-item">影视源</t-radio>
                <t-radio v-model="formData.clearSeletct.iptv" allow-uncheck class="radio-item">电视源</t-radio>
                <t-radio v-model="formData.clearSeletct.analyze" allow-uncheck class="radio-item">解析源</t-radio>
                <t-radio v-model="formData.clearSeletct.drive" allow-uncheck class="radio-item">网盘源</t-radio>
                <t-radio v-model="formData.clearSeletct.history" allow-uncheck class="radio-item">历史数据「含解析、搜索」</t-radio>
                <t-radio v-model="formData.clearSeletct.star" allow-uncheck class="radio-item">收藏</t-radio>
                <t-radio v-model="formData.clearSeletct.thumbnail" allow-uncheck class="radio-item">缩略图
                  <span class="title">「{{ formData.size.thumbnail }}MB」</span>
                </t-radio>
                <t-radio v-model="formData.clearSeletct.cache" allow-uncheck class="radio-item">缓存
                  <span class="title">「{{ formData.size.cache }}MB」</span>
                </t-radio>
                <t-button size="small" block ghost @click="clearData">清理</t-button>
              </t-collapse-panel>
            </t-collapse>
          </div>
          <div class="action">
            <div class="action-item"></div>
          </div>
        </div>
        <div class="data-item">
          <span class="separator"></span>
          <p class="title">同步盘</p>
          <p class="content">1. 因不收集用户的数据, 可以选择同步盘作为配置文件保存服务</p>
          <p class="content">2. 内置webdav作为同步盘服务, 推荐坚果云</p>
          <div class="config"> 
            <t-collapse>
              <t-collapse-panel value="0" header="配置同步盘参数">
                <template #headerRightContent>
                  <t-space size="small">  
                    <t-button size="small" @click.stop="saveWebdev">保存</t-button>
                    <t-button theme="default" size="small" @click.stop="checkWebdev">校验</t-button>
                  </t-space>
                </template>
                <t-input label="云地址：" v-model="formData.url" class="input-item"></t-input>
                <t-input label="用户名：" v-model="formData.username" class="input-item"></t-input>
                <t-input label="授权码：" v-model="formData.password" type="password" class="input-item"></t-input>
              </t-collapse-panel>
            </t-collapse>
          </div>
          <div class="action">
            <div class="action-item">
              <t-popconfirm content="云端数据将被覆盖,确认操作吗" placement="bottom"  @confirm="rsyncRemote">
                <t-button theme="default" class="btn-2">同步数据到帐号</t-button>
              </t-popconfirm>
              <t-popconfirm content="本地数据将被清除,确认操作吗" placement="bottom"  @confirm="rsyncLocal">
                <t-button theme="default" class="btn-2">云数据覆盖本地</t-button>
              </t-popconfirm>
            </div>
          </div>
        </div>
      </div>
    </template>
  </t-dialog>
</template>

<script setup lang="ts">
import { useIpcRenderer } from '@vueuse/electron';
import { useEventBus } from '@vueuse/core';
import _ from 'lodash';
import { MessagePlugin } from 'tdesign-vue-next';
import { ref, watch, reactive } from 'vue';
import { createClient } from "webdav";

import { analyze, analyzeHistory, channelList, history, iptv, setting, sites, star, searchHistory, drive } from '@/lib/dexie';
import zy from '@/lib/utils/tools';

const ipcRenderer = useIpcRenderer();
const remote = window.require('@electron/remote');
const win = remote.getCurrentWindow();

const filmEmitReload = useEventBus('film-reload');
const iptvEmitReload = useEventBus('iptv-reload');
const analyzeEmitReload = useEventBus('analyze-reload');
const historyEmitReload = useEventBus('history-reload');
const bingeEmitReload = useEventBus('binge-reload');
const driveEmitReload = useEventBus('drive-reload');

const props = defineProps({
  visible: {
    type: Boolean,
    default: false,
  },
  webdev: {
    type: Object,
    default: () => {
      return {};
    },
  }
});
const formVisible = ref(false);
const formData = reactive({
  url: '',
  username: '',
  password: '',
  easyConfig: {
    type: 0,
    url: ''
  },
  importData: {
    type: 'remote',
    remoteImpoUrl: '',
    localImpoFile: [],
  },
  clearSeletct: {
    sites: false,
    iptv: false,
    analyze: false,
    drive: false,
    history: false,
    star: false,
    setting: false,
    cache: false,
    thumbnail: false,
  },
  size: {
    cache: '',
    thumbnail: '',
  },
  exportSeletct: {
    sites: false,
    iptv: false,
    analyze: false,
    drive: false,
    history: false,
    star: false,
    setting: false,
  }
});
const clientWebdev = ref(null);

const emit = defineEmits(['update:visible']);

watch(
  () => formVisible.value,
  (val) => {
    emit('update:visible', val);
  },
);
watch(
  () => props.visible,
  (val) => {
    formVisible.value = val;
    if (val) {
      getCacheSize();
      getThumbnailSize();
    };
  },
);
watch(
  () => props.webdev,
  (val) => {
    formData.url = val.webdevUrl;
    formData.username = val.webdevUsername;
    formData.password = val.webdevPassword;
  }
)

// 数据判断
const hasTrueValue = (obj) => {
  console.log(obj)
  for (const key in obj) {
    if (obj.hasOwnProperty(key) && obj[key] === true) {
      return true;
    };
  };
  return false;
};

// 初始化数据库
const initDB = async(data) => {
  if (typeof data !== 'object') return;
  console.log(data);

  try {
    // 添加数据
    if (_.has(data, "sites")) {
      await sites.clear();
      sites.bulkAdd(data["sites"]);
    };

    if (_.has(data, "iptv")) {
      await iptv.clear();
      iptv.bulkAdd(data["iptv"]);
    };

    if (_.has(data, "channel")) {
      await channelList.clear();
      channelList.bulkAdd(data["channel"]);
    };

    if (_.has(data, "analyze")) {
      await analyze.clear();
      analyze.bulkAdd(data["analyze"]);
    };

    if (_.has(data, "drive")) {
      await drive.clear();
      drive.bulkAdd(data["drive"]);
    };

    if (_.has(data, "setting")) {
      await setting.clear();
      setting.bulkAdd(data["setting"]);
    };

    if (_.has(data, "history")) {
      await history.clear();
      history.bulkAdd(data["history"]);
    };

    if (_.has(data, "analyzeHistory")) {
      await analyzeHistory.clear();
      analyzeHistory.bulkAdd(data["analyzeHistory"]);
    };

    if (_.has(data, "searchHistory")) {
      await searchHistory.clear();
      searchHistory.bulkAdd(data["searchHistory"]);
    };

    if (_.has(data, "star")) {
      await star.clear();
      star.bulkAdd(data["star"]);
    };
    MessagePlugin.success('导入成功, 请等待一秒后加载数据！');
    setTimeout(() => {
      window?.location.reload();
    }, 1000);
  } catch (error) {
    MessagePlugin.error(`导入失败：${error}`);
    console.log(error);
  }

  formVisible.value = false;
};

// 一键配置
const easyConfig = async() => {
  const { url, type } = formData.easyConfig;
  if (!url) {
    MessagePlugin.warning('请输入一键配置地址');
    return;
  };
  const config = await zy.getConfig(url).catch((error) => {
    MessagePlugin.error(`请求一键配置地址失败：${error}`);
  });
  console.log(typeof config);
  if (typeof config !== 'object') return;

  console.log(config);
  const data = {};
  try {
    // 添加数据
    const defaultObject = {
      defaultSite: '',
      defaultIptv: '',
      defaultAnalyze: '',
      defaultDrive: ''
    };

    if (type === 0) {
      if (_.has(config, "sites")) {
        data["sites"] = config.sites.data;
        defaultObject.defaultSite = config.sites.default;
      };
      if (_.has(config, "iptv")) {
        data["iptv"] = config.iptv.data;
        defaultObject.defaultIptv = config.iptv.default;
      };
      if (_.has(config, "analyze")) {
        data["analyze"] = config.analyze.data;
        defaultObject.defaultAnalyze = config.analyze.default;
      };
      if (_.has(config, "drive")) {
        data["drive"] = config.drive.data;
        defaultObject.defaultDrive = config.drive.default;
      };
      setting.update(defaultObject);
    } else {
      if (_.has(config, "sites")) {
        const formatType = (type)=>{
          switch (type) {
            case 1:
              return 2;
            case 2:
              return 1;
            case 3:
              return 6;
          }
        }
        const formatGroup = (group)=>{
          switch (type) {
            case 1:
              return 'drpy';
            case 2:
              return 'tvbox';
            case 3:
              return 'hipy';
          }
        }
        data["sites"] = config.sites
          .filter((item) => item.type === 0 || item.type === 1) // 先过滤掉不需要的数据
          .map((item) => ({
            key: item.key,
            name: item.name,
            type: formatType(type),
            api: item.api,
            group: formatGroup(type),
            search: _.has(item, "searchable") ? item.searchable : 0,
            isActive: true,
            status: true,
          }));
      };
      if (_.has(config, "lives")) {
        // 提取group为"redirect"的channels
        const redirectChannels = config.lives.filter((live) => live.group === "redirect")[0].channels;

        // 解密URL中的ext并组合成iptv对象
        const iptv = redirectChannels.map((channel) => {
          const ext = channel.urls[0].split('&ext=')[1];
          const decodedExt = Buffer.from(ext, 'base64').toString('utf-8');
          
          return {
            name: channel.name,
            type: "remote",
            isActive: true,
            url: decodedExt
          };
        });
        data["iptv"] = iptv;
      };
      if (_.has(config, "parses")) {
        const analyze = config.parses
          .filter((item) => item.url && item.url.startsWith("http")) // 先过滤掉不需要的数据
          .map((item) => ({
            name: item.name,
            url: item.url,
            isActive: true,
          }));
        data["analyze"] = analyze;
      };
      if (_.has(config, "drives")) {
        const drives = config.drives
          .filter((item) => item.type === 'alist' || !item.type) // 先过滤掉不需要的数据
          .map((item) => ({
            name: item.name,
            server: item.server,
            startPage: item.startPage ? item.startPage : '',
            search: item.search ? item.search : false,
            headers: item.headers ? item.headers : null,
            params: item.params ? item.params : null,
            isActive: true,
          }));
        data["drive"] = drives;
      };
    };
    console.log(data)

    initDB(data);
  } catch (error) {
    MessagePlugin.error(`一键配置失败：${error}`);
    console.log(error);
  }
};

// 配置导入
const importData = async() => {
  const { type, remoteImpoUrl, localImpoFile } = formData.importData;
  console.log(localImpoFile.length === 0)
  if ((type === 'remote' && !remoteImpoUrl)  || (type === 'local') && (localImpoFile.length === 0)) {
    MessagePlugin.warning('请选择或者填写数据');
    return;
  };

  if (type === 'remote') {
    await importFromRemote(remoteImpoUrl);
  } else {
    await importFromLocal(localImpoFile[0].raw);
  };
};

// 远端导入
const importFromRemote = async(url) => {
  const config = await zy.getConfig(url).catch((error) => {
    MessagePlugin.error(`请求地址失败：${error}`);
  });
  await initDB(config);
}

// 本地导入
const importFromLocal = (file) => {
  const reader = new FileReader();
  reader.readAsText(file);
  reader.onload = (resultFile) => {
    const pointsTxt = resultFile.target.result;
    try {
      const json = JSON.parse(String(pointsTxt));
      initDB(json);
    } catch (err) {
      MessagePlugin.error(`导入失败：${err}`);
    }
  }
}

// 文件事件
const requestMethod = (file) => {
  return new Promise((resolve) => {
    // file.percent 用于控制上传进度，如果不希望显示上传进度，则不对 file.percent 设置值即可。
    // 如果代码规范不能设置 file.percent，也可以设置 files
    file.percent = 0;
    const timer = setTimeout(() => {
      // resolve 参数为关键代码
      resolve({ status: 'success', response: { url: file.name } });
      file.percent = 100;
      clearTimeout(timer);
    }, 1000);
  });
};

// 读取数据库数据
const jsonFromDB = async(all=false) => {
  const arr = {};
  try {
    if (formData.exportSeletct.analyze || all) {
      const analyze_data = await analyze.all();
      arr["analyze"] = [...analyze_data];
    }
    
    if (formData.exportSeletct.iptv || all) {
      const iptv_data = await iptv.all();
      arr["iptv"] = [...iptv_data];
      const channel_data = await channelList.all();
      arr["channel"] = [...channel_data];
    }

    if (formData.exportSeletct.sites || all) { 
      const sites_data = await sites.all();
      arr["sites"] = [...sites_data];
    }

    if (formData.exportSeletct.drive || all) { 
      const drive_data = await drive.all();
      arr["drive"] = [...drive_data];
    }
    
    if (formData.exportSeletct.setting || all) { 
      const setting_data = await setting.all();
      arr["setting"] = [...setting_data];
    }

    if (formData.exportSeletct.history || all) { 
      const analyzeHistory_data = await analyzeHistory.all();
      arr["analyzeHistory"] = [...analyzeHistory_data];
      const history_data = await history.all();
      arr["history"] = [...history_data];
      const searchHistory_data = await searchHistory.all();
      arr["searchHistory"] = [...searchHistory_data];
    }

    if (formData.exportSeletct.star || all) { 
      const star_data = await star.all();
      arr["star"] = [...star_data];
    }

    return arr;
  } catch (err) {
    console.log('data export fail');
    MessagePlugin.success('data export fail');
    return {};
  }
}

// 导出
const setExport = async() => {
  if (!hasTrueValue(formData.exportSeletct)) {
    MessagePlugin.warning('请选择需要导出的类型');
    return;
  };
  const arr = await jsonFromDB();
  const str = JSON.stringify(arr, null, 2);
  const blob = new Blob([str], { type: 'text/plain;charset=utf-8' });
  const reader = new FileReader();
  reader.onload = () => {
    const result: ArrayBuffer = reader.result as ArrayBuffer;
    const buffer = Buffer.from(result);
    remote.dialog
      .showSaveDialog(remote.getCurrentWindow(), {
        defaultPath: 'config.json',
        filters: [{ name: 'JSON Files', extensions: ['json'] }],
      })
      .then((saveDialogResult) => {
        if (!saveDialogResult.canceled) {
          const { filePath } = saveDialogResult;
          const fs = remote.require('fs');
          fs.writeFile(filePath, buffer, 'utf-8', (err) => {
            if (err) {
              console.error('Failed to save file:', err);
              MessagePlugin.error('Failed to save file');
            } else {
              console.log('File saved successfully');
              MessagePlugin.success('File saved successfully');
            }
          });
        }
      })
      .catch((err) => {
        console.error('Failed to open save dialog:', err);
        MessagePlugin.error('Failed to open save dialog');
      });
  };
  reader.readAsArrayBuffer(blob);
};

//  获取 cache 大小
const getCacheSize = async () => {
  const { session } = win.webContents;
  const getSize = await session.getCacheSize() / 1024 / 1024;
  const formatToMb = getSize.toFixed(2);
  formData.size.cache = formatToMb;
};

//  获取 thumbnail 文件夹大小
const getThumbnailSize = () => {
  ipcRenderer.removeAllListeners("tmpdir-manage-size"); // 移除之前的事件监听器
  ipcRenderer.send('tmpdir-manage', 'size', 'thumbnail');
  ipcRenderer.on("tmpdir-manage-size", (_, data) => {
    const getSize = data / 1024 / 1024;
    const formatToMb = getSize.toFixed(2);
    formData.size.thumbnail = formatToMb;
  });
};

// 清理缓存
const clearData = async() => {
  if (!hasTrueValue(formData.clearSeletct)) {
    MessagePlugin.warning('请选择需要清理的类型');
    return;
  };
  // 清空sites数据的函数
  const clearSitesData = async () => {
    await sites.clear();
    await setting.update({
      defaultSite: null,
    });
    filmEmitReload.emit('film-reload');
  };

  // 清空iptv数据的函数
  const clearIptvData = async () => {
    await iptv.clear();
    await channelList.clear();
    await setting.update({
      defaultIptv: null,
    });
    iptvEmitReload.emit('iptv-reload');
  };

  // 清空analyze数据的函数
  const clearAnalyzeData = async () => {
    await analyze.clear();
    await setting.update({
      defaultAnalyze: null,
    });
    analyzeEmitReload.emit('analyze-reload');
  };

  // 清空drive数据的函数
  const clearDriveData = async () => {
    await drive.clear();
    await setting.update({
      defaultDrive: null,
    });
    driveEmitReload.emit('drive-reload');
  };

  // 清空history数据的函数
  const clearHistoryData = async () => {
    await history.clear();
    await analyzeHistory.clear();
    await searchHistory.clear();
    historyEmitReload.emit('history-reload');
  };

  // 清空binge数据的函数
  const clearBingeData = async () => {
    await star.clear();
    bingeEmitReload.emit('binge-reload');
  };

  // 清空cache数据的函数
  const clearCacheData = async () => {
    const { session } = win.webContents;
    await session.clearCache();
    await getCacheSize();
  };

  // 清空 缩略图的函数
  const clearThumbnail = async () => {
    await ipcRenderer.send('tmpdir-manage', 'rmdir', 'thumbnail');
    iptvEmitReload.emit('iptv-reload');
  }

  // 清空数据的映射对象
  const clearDataFunctions = {
    sites: clearSitesData,
    iptv: clearIptvData,
    analyze: clearAnalyzeData,
    drive: clearDriveData,
    history: clearHistoryData,
    binge: clearBingeData,
    cache: clearCacheData,
    thumbnail: clearThumbnail,
  };

  try {
    for (const key in formData.clearSeletct) {
      if (formData.clearSeletct[key]) {
        // 根据formData值执行对应的清空数据函数
        const clearDataFunction = clearDataFunctions[key];
        if (clearDataFunction) {
          clearDataFunction();
        }
      }
    }
    MessagePlugin.success('清理成功');
  } catch (err) {
    MessagePlugin.error(`清理失败:${err}`);
  }
}

// 初始化
const initWebdav = async() => {
  if (!formData.url && !formData.username && !formData.password) {
    MessagePlugin.warning('请补全同步盘配置信息');
    return;
  };
  clientWebdev.value = await createClient(
    formData.url,
    {
      username: formData.username,
      password: formData.password
    }
  );
  if (await clientWebdev.value.exists("/zyplayer") === false) {
    await clientWebdev.value.createDirectory("/zyplayer");
  };
};

// 保存
const saveWebdev = async() => {
  try {
    await setting.update({
      webdevUrl: formData.url,
      webdevUsername: formData.username,
      webdevPassword: formData.password,
    });
    MessagePlugin.success('保存成功');
  } catch (err) {
    MessagePlugin.error(`保存失败:${err}`)
  };
};

// 校验
const checkWebdev = async() => {
  try {
    if (!clientWebdev.value) await initWebdav();
    if (!formData.url && !formData.username && !formData.password) return;
    await clientWebdev.value.getDirectoryContents("/");
    MessagePlugin.success('连接成功');
  } catch (err) {
    MessagePlugin.error(`连接失败:${err}`)
  };
}

// 覆盖远端
const rsyncRemote = async() => {
  try {
    if (!clientWebdev.value) await initWebdav();
    if (!formData.url && !formData.username && !formData.password) return;
    const str = await jsonFromDB(true);
    const formatToJson = JSON.stringify(str);
    console.log(formatToJson)
    await clientWebdev.value.putFileContents("/zyplayer/config.json", formatToJson, { overwrite: false });
    MessagePlugin.success('传输成功');
  } catch (err) {
    MessagePlugin.error(`传输失败:${err}`);
    console.error(err);
  };
}

// 覆盖本地
const rsyncLocal = async() => {
  try {
    if (!clientWebdev.value) await initWebdav();
    if (!formData.url && !formData.username && !formData.password) return;
    const str: string = await clientWebdev.value.getFileContents("/zyplayer/config.json", { format: "text" });
    const formatToJson = JSON.parse(str);
    console.log(formatToJson)
    initDB(formatToJson);
    MessagePlugin.success('获取云端文件成功');
  } catch (err) {
    MessagePlugin.error(`获取文件失败:${err}`);
    console.error(err);
  };
}

</script>
<style lang="less" scoped>
.data-dialog-container {
  max-height: 430px;
  .data-item {
    .separator {
      border: 0.1rem solid var(--td-brand-color);
      height: 0.6rem;
      border-radius: var(--td-radius-default);
      display: inline-block;
    }
    .title {
      color: var(--td-brand-color);
      margin-left: 5px;
      display: inline-block;
      text-align: left;
      font-weight: 500;
    }
    .content {
      font-size: 12px;
    }
    .config {
      .input-item {
        margin-bottom: var(--td-comp-margin-s);
      }
      :deep(.t-collapse-panel__content) {
        padding: var(--td-pop-padding-m);
      }
      :deep(.t-form__controls-content .t-input) {
        background: aquamarine !important;
      }
      .input-item,
      :deep(.t-upload__dragger) {
        width: 100%;
      }
    }
    .action {
      .action-item {
        margin-top: var(--td-comp-margin-s);
        display: flex;
        justify-content: space-between;
        .btn-2 {
          width: 49%;
        }
        .btn-3 {
          width: 33%;
        }
      }
    }
  }
}
</style>
