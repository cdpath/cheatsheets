参考书：
[tmux Taster](http://book.douban.com/subject/26296805/)

## 基本操作
下面所有**内部命令**和**快捷键**都需要先按下 `<P>`。

目的|外部命令|内部命令(`<P>`)|快捷键(`<P>`)|说明
---|---|---|---|---
断开 Session|||`d`|
分割 window |`tmux split-window -t foo:1`|||分割窗口就意味着创建两个 pane
调整 pane 大小||`:resize-panel -t {窗口号} -D {大小}`||
加大 pane（向左）||`:resizep -L`|`<`|
加大 pane（向上）||`:resizep -U`|`+`|
均分 pane（水平）||`:select-layout even-horizontal`||
均分 pane（垂直）||`:select-layout even-vertical`||
显示 pane 编号||`:display-panes`|`q`|
跳转到上个 pane||`:last-pane`|`;`|
将 pane 全屏||`:resize-pane -Z`|`z`|跳转 pane 或者重复快捷键即可退出
pane -> window ||`:break-pane`|`!`|
window -> pane ||`:join-pane -s {source_window} -t {target_window}`||
旋转 pane 布局（顺时针）|||`}`|
旋转 pane 布局（逆时针）|||`{`|
重新分配 pane 布局|||`空格`|tmux 会自动改变布局
** Window **|
新建 window||`:new-window`|`c`|tmux 底部状态栏会显示 window 信息
重命名 window||`:rename-window -t {窗口号｜旧名字} {新名字}`|`,`|默认是新窗口的进程名
显示 window 编号||`:choose-window`|`w`|
下一个 Window|||`n`|
上一个 Window|||`p`|
跳转到指定 Window|||`数字`|输入 Window 号
搜索并跳转到 Window||`find-window`|`f`|可以搜索窗口名，或者窗口内容
关闭 Window（发送 EOF）|||`Ctrl-D`|不需要 `<P>`
关闭 Window（当前窗口） ||`kill-window`||
关闭 Window（只保留当前窗口） ||`kill-window -a`||当前窗口保留，其他都删除
关闭 Window（关闭指定窗口） ||`kill-window -t 编号`||关闭窗口号对应的窗口
重新布局 Window (更改位置）||`move-window -t 编号`||**未成功！**
重新布局 Window (交换位置）||`swap-window -s 编号 -t 编号`||
跨 Session 重新布局 Window |`tmux move-window -s foo:1 -t bar:2`|`move-window -t bar:2`|`.`|
跨 Session 共享 Window ||`:link-window -s foo: 1 -t bar:2`||不指定 `-s` 就使用当前Session当前Window；使用 `-k` 强行共享
**拷贝粘贴**|
进入 Copy Mode||`:copy-mode`|`[`|
退出|||`q` 或 `回车`|Linux: `q` 或 `ESC`
选择过程|||`空格` 到 `回车`|Linux: `^空格` 到 `ESC-w`
查看剪贴板缓存||`:list-buffers`|`#`|
查看最新剪贴板缓存||`:show-buffer`||
选择内容去粘贴||`:choose-buffer`|`=`（已被占用）|
粘贴||`:paste-buffer`|`]`|
保存剪贴板内容到文件||`:save-buffer ~/path`||

## `.bashrc`
目的|方法
---|---
新建 Session |`alias tat="tmux new-session -As $(basename $PWD｜tr . -)"`
进入已有 Session | `function tmuxopen() { tmux attach -t $1 }`
删除指定 Session | `function tmuxkill() {tmux kill-session -t $1 }`
删除所有 Session | `alias tmuxkillall="tmux ls｜cut -d : -f 1｜xargs -I {} tmux kill-session -t {}"`

## 自动化
1\. **`session -> window -> vim -> input`**
`tmux send-keys -t 1 "vim" "C-m" "i" "输入内容咯"`
- `-t 1` 表示给*最近活跃的 session` 的 1 号 pane 发送按键。
- `-t foo:1` 具体指定目标

2\. **结合 Shell**
`<P>:run-shell [-b] [-t {pane_id}] "{shell_command}"`
- `-b` 将 shell 放到后台运行
- `-t pane号` 将 shell 输出放到指定 pane 显示

可以用来**自定义按键**：
`bind-key e select-pane -L \; run-shell "ls -la && ls ~/Desktop"`

甚至是**自适应按键**：
`bind-key u if-shell "test $(ls | grep Dropbox | wc -l) -gt 0" "display-message 'Folder does not exist'" "display-message 'Folder exists'"`


## 特殊功能
1\. 自动重复 `<P>` (Repeatable Keys)
`-r`
需要连续进行多个 tmux 操作时，只需要按一次 `<P>`，不用每个命令都由 `<P>` 开始。

2\. 多 pane 同步输入
**注意，同一 window 内的全部 pane 都会同步**
`<P>:setw synchronize-panes`
`<P>:setw synchronize-panes off`


## 其他问题
1\. 在 tmux 之前运行的进程怎么放到 tmux 中？

只能在 Linux 下借助 `Reptyr` 实现，具体步骤：

1. 将进程放入后台：`^Z`
2. 和父进程 disown：`disown <进程名>`
3. `tmux`
4. `reptyr $(pgrep <进程名>)` 或者 `reptyr <PID>`

2\. 操作系统重启了怎么恢复状态？

借助 tmux 插件，[tmux-resurrect](https://github.com/ tmux-plugins/tmux-resurrect)

3\. tmux 环境和 bash 环境的冲突

参考 [PS1 的问题](http://stackoverflow.com/questions/21005966/tmux-prompt-not-following-normal-bash-prompt-ps1-w)

4\. `^R` 反向搜索历史在 tmux 中无法使用：

在 `.tmux.conf` 中绑定按键，
```bash
bindkey '^R' history-incremental-search-backward
bindkey '^A' beginning-of-line
bindkey '^E' end-of-line
```

## workflow
> we weren’t developing our applications in an environment that accurately represented the live server environment.

<placeholder>
