<template>
  <div id="app">
    <router-view />
  </div>
</template>

<script>
export default {
  name: 'App',
  mounted() {
    this.initDataBase()
  },
  methods: {
    async initDataBase() {
      let db;
      let dbName = 'xdd-server';

      try {
        // 首先尝试打开数据库以获取当前版本号
        db = await this.openDatabase(dbName);
        const currentVersion = db.version;
        db.close(); // 关闭数据库，准备升级版本

        // 再次打开数据库，这次指定新的版本号进行升级
        const newVersion = currentVersion + 1;
        const request = window.indexedDB.open(dbName, newVersion);

        request.onupgradeneeded = (event) => {
          const db = event.target.result;
          if (!db.objectStoreNames.contains('users')) {
            const objectStore = db.createObjectStore('users', { keyPath: 'id' });
            // 创建索引
            objectStore.createIndex('username', 'username', { unique: true });
            objectStore.createIndex('password', 'password', { unique: false });
            objectStore.createIndex('nickName', 'nickName', { unique: false });

            // 向`users`对象存储添加初始数据
            objectStore.transaction.oncomplete = (event) => {
              const userObjectStore = db.transaction('users', 'readwrite').objectStore('users');
              userObjectStore.add({id: '1', username: 'admin', password: 'admin', nickName: '小叮当'});
              userObjectStore.add({id: '2', username: 'editor', password: 'editor', nickName: '普通用户'});
            };
          }
        };

        request.onsuccess = (event) => {
          console.log('数据库升级成功，数据写入成功');
        };

        request.onerror = (event) => {
          throw new Error('数据库打开失败');
        };
      } catch (error) {
        console.error('初始化数据库失败:', error);
      }
    },
    async openDatabase(dbName) {
      return new Promise((resolve, reject) => {
        const request = window.indexedDB.open(dbName);
        request.onerror = () => reject(new Error('数据库打开失败'));
        request.onsuccess = (event) => resolve(event.target.result);
      });
    }
  }
}
</script>
