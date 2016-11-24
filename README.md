# xiaopin.github.io


### Use


#### 1 Recursive parameters are used (--recursive)
```bash
git clone -b hexo https://github.com/xiaopin/xiaopin.github.io.git --recursive
cd xiaopin.github.io/hexo/
npm install
```

#### 2 The second method first clone the parent project, and then initialize the Submodule.

##### 2.1 clone hexo branch
```bash
git clone -b hexo https://github.com/xiaopin/xiaopin.github.io.git
cd xiaopin.github.io/hexo/
npm install
```

##### 2.2 clone submodule
```bash
cd ../
git submodule init
git submodule update
```
