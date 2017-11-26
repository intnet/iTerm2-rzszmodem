#!/bin/bash



<<'iTerm2-rzszmodem'

----
## Setup

 * Install lrzsz on MACOS, just like: ```brew install lrzsz```
 * 【用 homebrew 为本机 MACOS 安装好 lrzsz】
 * Save this file as ```/usr/local/bin/rzszmodem``` then chmod 755
 * 【将此文件另存为 /usr/local/bin/rzszmodem，并分配权限 755】
 * Set up iTerm2/Preferences/Profiles/Default/Advanced/Triggers like so:
 * 【在 iTerm2 偏好设置 Profiles/Default/Advanced/Triggers 中，添加如下规则】

```
  Regular expression: rz waiting to receive.\*\*B0100
  Action: Run Silent Coprocess
  Parameters: /usr/local/bin/rzszmodem rz

  Regular expression: \*\*B00000000000000
  Action: Run Silent Coprocess
  Parameters: /usr/local/bin/rzszmodem sz
```


## Usage

**Send a file to remote: 【上传文件到远程主机】**

 * type ```rz -be``` on the remote 【在远程机上运行 rz -be】
 * select a file on the local machine 【在弹出框中选择一个本机文件】

**Receive a file from remote: 【下载文件到本地】**

 * type ```sz -be filename``` on the remote 【在远程机上运行 sz -be】
 * select the folder on the local machine 【在弹出框中选择本地保存目录】

iTerm2-rzszmodem



<<'mmastrac/iterm2-zmodem'

----
### See: [mmastrac iterm2-zmodem](https://github.com/mmastrac/iterm2-zmodem)

iterm2-send-zmodem.sh
```bash
osascript -e 'tell application "iTerm2" to version' > /dev/null 2>&1 && NAME=iTerm2 || NAME=iTerm
if [[ $NAME = "iTerm" ]]; then
    FILE=`osascript -e 'tell application "iTerm" to activate' -e 'tell application "iTerm" to set thefile to choose file with prompt "Choose a file to send"' -e "do shell script (\"echo \"&(quoted form of POSIX path of thefile as Unicode text)&\"\")"`
else
    FILE=`osascript -e 'tell application "iTerm2" to activate' -e 'tell application "iTerm2" to set thefile to choose file with prompt "Choose a file to send"' -e "do shell script (\"echo \"&(quoted form of POSIX path of thefile as Unicode text)&\"\")"`
fi
if [[ $FILE = "" ]]; then
    echo Cancelled.
    # Send ZModem cancel
    echo -e \\x18\\x18\\x18\\x18\\x18
    sleep 1
    echo
    echo \# Cancelled transfer
else
    /usr/local/bin/sz "$FILE" -e -b
    sleep 1
    echo
    echo \# Received $FILE
fi
```

iterm2-recv-zmodem.sh
```bash
osascript -e 'tell application "iTerm2" to version' > /dev/null 2>&1 && NAME=iTerm2 || NAME=iTerm
if [[ $NAME = "iTerm" ]]; then
    FILE=`osascript -e 'tell application "iTerm" to activate' -e 'tell application "iTerm" to set thefile to choose folder with prompt "Choose a folder to place received files in"' -e "do shell script (\"echo \"&(quoted form of POSIX path of thefile as Unicode text)&\"\")"`
else
    FILE=`osascript -e 'tell application "iTerm2" to activate' -e 'tell application "iTerm2" to set thefile to choose folder with prompt "Choose a folder to place received files in"' -e "do shell script (\"echo \"&(quoted form of POSIX path of thefile as Unicode text)&\"\")"`
fi

if [[ $FILE = "" ]]; then
    echo Cancelled.
    # Send ZModem cancel
    echo -e \\x18\\x18\\x18\\x18\\x18
    sleep 1
    echo
    echo \# Cancelled transfer
else
    cd "$FILE"
    /usr/local/bin/rz -E -e -b
    sleep 1
    echo
    echo
    echo \# Sent \-\> $FILE
fi
```

mmastrac/iterm2-zmodem



<<'aurora/iterm2-zmodem'

----
### See: [aurora iterm2-zmodem](https://github.com/aurora/iterm2-zmodem)

iterm2-zmodem
```bash
# usage
if [[ $1 != "sz" && $1 != "rz" ]]; then
    echo "usage: $0 sz|rz"
    exit
fi

# send Z-Modem cancel sequence
function cancel {
    echo -e \\x18\\x18\\x18\\x18\\x18
}

# send notification using growlnotify
function notify {
    local msg=$1
    
    if command -v growlnotify >/dev/null 2>&1; then
        growlnotify -a /Applications/iTerm.app -n "iTerm" -m "$msg" -t "File transfer"
    else
        echo "# $msg" | tr '\n' ' '
    fi
}

#setup
[[ $LRZSZ_PATH != "" ]] && LRZSZ_PATH=":$LRZSZ_PATH" || LRZSZ_PATH=""

PATH=$(command -p getconf PATH):/usr/local/bin$LRZSZ_PATH
ZCMD=$(
    if command -v $1 >/dev/null 2>&1; then
        echo "$1"
    elif command -v l$1 >/dev/null 2>&1; then
        echo "l$1"
    fi
)

# main
if [[ $ZCMD = "" ]]; then
    cancel
    echo

    notify "Unable to find Z-Modem tools"
    exit
elif [[ $1 = "rz" ]]; then
    # receive a file
    DST=$(
        osascript \
            -e "tell application \"iTerm\" to activate" \
            -e "tell application \"iTerm\" to set thefile to choose folder with prompt \"Choose a folder to place received files in\"" \
            -e "do shell script (\"echo \"&(quoted form of POSIX path of thefile as Unicode text)&\"\")"
    )
    
    if [[ $DST = "" ]]; then
        cancel
        echo 
    fi

    cd "$DST"
    
    notify "Z-Modem started receiving file"

    $ZCMD -e -y
    echo 

    notify "Z-Modem finished receiving file"
else
    # send a file
    SRC=$(
        osascript \
            -e "tell application \"iTerm\" to activate" \
            -e "tell application \"iTerm\" to set thefile to choose file with prompt \"Choose a file to send\"" \
            -e "do shell script (\"echo \"&(quoted form of POSIX path of thefile as Unicode text)&\"\")"
    )

    if [[ $SRC = "" ]]; then
        cancel
        echo 
    fi

    notify "Z-Modem started sending
$SRC"

    $ZCMD -e "$SRC"
    echo 

    notify "Z-Modem finished sending
$SRC"
fi
```

aurora/iterm2-zmodem



<<'AppleScript'

----
### See: [AppleScript osascript](https://stackoverflow.com/questions/4309087/cancel-button-on-osascript-in-a-bash-script)

```bash
WORKDIR="$(dirname "$0")/"

STARTUPFILE="$(/usr/bin/osascript -e "tell application \"System Events\" to activate" -e "tell application \"System Events\" to set thefile to choose file with prompt \"Choose something here\"" -e "do shell script (\"echo \"&(quoted form of POSIX path of thefile as Unicode text)&\"\")")"


if [[ $* = *"Option 1 from Applescript"* ]]; then

cp -R "$STARTUPFILE/" "somewhere else"

do other stuff with "$STARTUPFILE...
```

AppleScript






<<'#BASHcode'
```bash
#BASHcode

function echoCancel {
  echo -e "\\x18\\x18\\x18\\x18\\x18" #         向远端发送取消信号
  [[ ${1} != '' ]] && echoNotify "${1}" #       如果带参数则稍后发送提示信息
}


function echoNotify {
  sMsg=${1//\\r/} #                             过滤待发送信息中的换行与回车
  sleep 0.1 #                                   似乎必须
  echo #                                        用于清掉远端 echo -n 的信息
  echo ": # :${sMsg//\\n/ }" #                  发给远端的信息会被当成指令执行
}


function choosePath { #                         弹出文件(夹)选择窗口
  if [[ ${1} != 'folder' && ${1} != 'file' ]]; then
    return #                                    只能选单个文件或单个目录
  fi
  # 激活 iTerm2 程序，弹出文件(夹)选择窗口，参数 2 用作窗口标题
  # 将用户选中的文件(夹)赋值给临时变量 tPath，然后转换成字符型绝对路径 sPath
  # 路径中可能包含空格符，后继处理要注意使用引号，但是这引号问题...
  echo $(osascript \
    -e 'tell application "iTerm2" to activate' \
    -e "tell application \"iTerm2\" to set tPath to choose ${1} with prompt \"${2:-Choose ${1}}\"" \
    -e 'set sPath to quoted form of POSIX path of tPath' \
    -e 'do shell script ("echo "&(sPath as Unicode text)&"")'
  )
}


function getZcmd {
  zPath="/usr/local/bin" #                      lrzsz 命令路径，可能不在环境变量中
  tPath=":${PATH}:" #                           添加之前先检查是否已经存在于 $PATH
  if [[ ${tPath} == ${tPath/:${zPath}:/:} ]]; then
    PATH=${zPath}:${PATH}
  fi
  if [[ ${1} == 'rz' ]]; then
    echo $(command -v 'sz') #                   检测本机是否已经安装 lrzsz
  elif [[ ${1} == 'sz' ]]; then #               本机待运行的命令与远端相呼应
    echo $(command -v 'rz') #                   远端 sz 对应本机 rz，反之亦然
  fi
}


function getSpath {
  if [[ ${1} == 'rz' ]]; then #                 远端运行 rz 时，本机选中一个文件，然后 sz 过去
    echo $(choosePath 'file')
  elif [[ ${1} == 'sz' ]]; then #               远端运行 sz 时，本机选择一个目录，用于存放 rz 的文件
    echo $(choosePath 'folder')
  fi
}



zCMD=$(getZcmd ${1})
if [[ ${zCMD} == '' ]]; then #                  检测本机是否已经安装 lrzsz
  echoCancel "Local lrzsz not found."
  exit
fi

sPath=$(getSpath ${1})
if [[ ${sPath} == '' ]]; then #                 用户没有选择文件(夹)，点了取消按钮
  echoCancel "Cancelled about ${1}" #           向远端发送取消指令，以退出其等待状态
  exit
fi

getZcmd #                                       为 lrzsz 命令设置好 PATH 环境变量
if [[ ${1} == 'rz' ]]; then #                   远端执行 receive 时，本机执行 send
  sz -be "${sPath}" #                           -b 二进制模式，-e 转义所有控制字符
else #                                          远端 send 则本机 receive，注意路径中的空格符
  cd "${sPath}" && rz -bey #                    -y 覆盖旧文件
fi
if [[ ${?} == '0' ]]; then #                    这里并没有出现非0的情况，为什么呢
  echoNotify "Completed ${1} at \"${sPath}\""
else
  echoNotify "Error on ${1} with \"${sPath}\""
fi

<<'EOF'
```
EOF

