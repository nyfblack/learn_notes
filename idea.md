# 1.search
- ctrl+o：打开可以重写的方法列表，重写父类的方法；
- ctrl+n：全局搜索类文件
- ctrl+r：替换内容
- ctrl+h：打开继承树；


# 2.others
- alt+insert：生成函数（get、set、构造器、toString...）；
- alt+enter：智能提示；
- ctrl+F9：开发过程中，修改html页面后，刷新页面，重新编译；
- shift+F6：重命名
- ctrl+alt+v：自动生成返回值
- ctrl+alt+l：自动格式化


# 3.创建一个web工程（使用maven）
- 创建一个项目：File->New Project->maven , 不选择模板->根据流程创建项目完成，然后进行如下配置；
- 打开 Project Structure（快捷键 Ctrl+Alt+Shift+S），Project不需要配置，编译目录保持默认即可；
- Moudles：无适配服务组件，需要添加。添加web；
- 提示 Web module没有包含artifact，根据提示创建；
- web描述文件和资源目录（默认）；
- Facts 表示当前项目的适配服务组件（默认）；
- Artifacts 描述当前发布信息（默认）
- ok 创建完成

