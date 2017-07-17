title: React Native 使用 SQLite 及 数据库升级策略
date: 2017-07-17 22:19:01
tags: 
- react-native
- sqlite
---

本文介绍 React Native 下结合 Redux 使用 SQLite 的方法，并介绍一种可行的数据库升级策略。

React Native 下使用 SQLite 使用 [react-native-sqlite-storage](https://github.com/andpor/react-native-sqlite-storage) 库，详细的配置及基本使用可参照该库的说明。

我的程序入口是 `App.js`，数据库相关的操作都在 `DataManager.js` 中, Redux 的 action 都放在 `actions.js` 中。

action 中定义与数据库处理相关的操作，比如添加某项数据：

``` javascript
export const addFavoriteLink = (linkItem, from) => {
    return function (dispatch) {
        return DataManager.getDb().transaction((tx) => {
            tx.executeSql(DataManager.SQL_INSERT_LINKITEM(linkItem)).then(([tx, results]) => {
                dispatch(loadFavoriteLink());
                dispatch(searchFavoriteLinkWithUrl(linkItem.URL));
            })
        })
    }
};
```
> transaction 的结果是一个 Promise， 返回后可以在业务逻辑中进行链式调用

`DataManager.getDb()` 获取数据库的实例，而实例是在 App.js 中 调用 `DataManager.initDB()` 初始化，代码如下：

``` javascript
componentDidMount() {
    InteractionManager.runAfterInteractions(() => {
        DataManager.initDB();
    });
}

exports.initDB = () => {
        _openDb().then((DB) => {
            createVideoTable(DB);
            createLinkTable(DB);
            _upgradeDatabase(DB);
        })
};

function _openDb() {
    return SQLite.openDatabase({name: DB_NAME, location: 'Library'}).then((db) => {
        this.db = db;
        return this.db;
    });
}
```

`reateVideoTable(DB)` 与 `createLinkTable(DB)` 可以看做对数据库的初始化, 然后执行数据库的升级策略，基本原理就是在一个数据表中单独存储 Version 的信息, 每次获取这个信息，如果需要升级就一步步的升级过去：

``` javascript
function _upgradeDatabase(db) {
    let versionExist = false;
    db.transaction((tx) => {
        tx.executeSql(SQL_SELECT_ALL_TABLE).then(([tx, results]) => {
            for (let i = 0; i < results.rows.length; i++) {
                let item = results.rows.item(i).name;
                if (item === 'VERSION') {
                    versionExist = true;
                    break;
                }
            }
            if (!versionExist) {
                db.transaction((tx) => {
                    tx.executeSql(SQL_CREATE_VERSION_TABLE);
                });

                db.transaction((tx) => {
                    tx.executeSql(SQL_INSERT_VERSION(0)).then(([tx, results]) => {
                        getVersionAndDoUpgrade(db)
                    })
                });
            } else {
                getVersionAndDoUpgrade(db)
            }
        })
    })
};

function getVersionAndDoUpgrade(db) {
    db.transaction((tx) => {
        tx.executeSql("SELECT VERSION FROM VERSION").then(([tx, results]) => {
            let version = results.rows.item(0).VERSION;
            if (version < CURRENT_SCHEMA_VERSION) {
                switch (version) {
                    case 0:
                        _initDatabase(db);
                        db.transaction((tx) => {
                            tx.executeSql(SQL_UPDATE_VERSION(CURRENT_SCHEMA_VERSION));
                        });
                        break;
                    default:
                        return;
                }
            }
        });
    });
}
```
