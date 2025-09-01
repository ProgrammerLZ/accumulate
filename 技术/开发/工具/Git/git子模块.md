## 拉取

```shell
git clone https://cnb.cool/zytec/agileleap/framework-java.git .
git submodule update --init --recursive
```



## 重命名子模块

```shell
git clone https://cnb.cool/zytec/agileleap/framework-java.git ./agileleap3
cd agileleap3
git submodule update --init -- service/agileleap
git submodule deinit -f service/agileleap
git submodule add https://cnb.cool/zytec/agileleap/deployment-java.git service/deployment
git rm --cached -r service/agileleap 2>/dev/null
git submodule update --init --recursive
git submodule absorbgitdirs
git submodule status

# 其他人需要更新
git submodule sync --recursive
git submodule update --init --recursive


```



