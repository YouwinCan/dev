## 安装
sudo apt install protobuf-compiler  
apt install python-protobuf  
## 加入动作监听进行编译
```
bazel build --experimental_action_listener=//tools/actions:generate_compile_commands_listener --copt=-w --cxxopt=-w --copt=-Wno-error  --cxxopt=-Wno-error --linkopt="-lbsd" --jobs=8 -c fastbuild
```
