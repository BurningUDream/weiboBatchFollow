# 微博批量关注 (转移关注列表)
第四次转世

# 起因
<img src="https://user-images.githubusercontent.com/26399680/47438989-1b910300-d7de-11e8-9957-b68888426d36.png" width="480"/>

# POWER BY GPT4.0 !!!

## 从 m.weibo.com 中提取
```javascript
(() => {
    let uid = config.uid, follows = []
    const pageQuery = (page = 1) => {
        let xhr = new XMLHttpRequest()
        xhr.onreadystatechange = () => {
            if(xhr.readyState == 4 && xhr.status == 200){
                let data = JSON.parse(xhr.responseText).data
                if(data.msg == '这里还没有内容') return console.log(JSON.stringify(follows))
                data.cards.forEach(card => follows.push(card.user.id))
                pageQuery(page + 1)
            }
        }
        xhr.open('GET', `/api/container/getSecond?luicode=10000011&lfid=100505${uid}&uid=${uid}&containerid=100505${uid}_-_FOLLOWERS&page=${page}`)
        xhr.send()
    }
    pageQuery()
})()
```

## 从正常网页微博中提取，在关注人页面，一直滚动直到输出结果
```javascript
// 创建一个全局的空数组，用于保存关注的人名单
const globalList = [];

// 创建一个计数器，用于记录全局列表的变化次数
let changeCount = 0;

// 添加一个函数，用于检查全局列表是否超过三次不再变化，并输出列表
function checkAndOutputList() {
  if (changeCount >= 10) {
    // 输出全局列表
    console.log('全局列表:', globalList);
    
    // 停止监听滚动事件，以防止重复输出
    window.removeEventListener('scroll', updateGlobalList);
  }
}

// 监听页面的滚动事件
function updateGlobalList() {
  // 找到具有 usercard 属性的 span 元素
  const elements = document.querySelectorAll('span[usercard]');
  
  // 遍历每个元素并提取其文本内容
  elements.forEach(element => {
    const content = element.innerText;
    
    // 检查内容是否已经在全局列表中
    if (!globalList.includes(content)) {
      // 将新的内容加入全局列表
      globalList.push(content);
      
      // 增加变化计数器
      changeCount++;
      
      // 重新设置计时器
      clearTimeout(outputTimeout);
      outputTimeout = setTimeout(checkAndOutputList, 1000);
    }
  });
}

// 初始化页面滚动事件监听
window.addEventListener('scroll', updateGlobalList);

// 初始化全局列表检查定时器
let outputTimeout = setTimeout(checkAndOutputList, 3000);
```

## 转移关注人，格式：[3027806214,5991116818,1847588883]
```javascript
(list => {
    const getRandomIndex = (length, usedIndexes) => {
        let randomIndex;
        do {
            randomIndex = Math.floor(Math.random() * length);
        } while (usedIndexes.includes(randomIndex));
        return randomIndex;
    }

    const usedIndexes = []; // 用于存储已使用的随机索引

    let index = -1, length = list.length
    const follow = uid => {
        let xhr = new XMLHttpRequest()
        xhr.onreadystatechange = () => {
            if(xhr.readyState == 4 && xhr.status == 200){
                let data = JSON.parse(xhr.responseText)
                if(data.code == 100000) {
                    console.log(`${index + 1}/${length}`, uid, data.data.fnick, '关注成功')
                } else if (data.code == 20062) {
                    console.log(`${index + 1}/${length}`, uid, `${data.msg}`)
                } else if (data.code == 100001) {
                    console.log(`${index + 1}/${length}`, uid, `${data.msg}`)
                } else {
                    console.log(`${index + 1}/${length}`, uid, `${data.code} - ${data.msg}`)
                }
            }
        }
        xhr.open('POST', `/aj/f/followed?ajwvr=6&__rnd=${Date.now()}`)
        xhr.setRequestHeader('Content-type', 'application/x-www-form-urlencoded')
        xhr.send(`uid=${uid}&objectid=&f=1&extra=&refer_sort=&refer_flag=1005050001_&location=page_100505_home&oid=${uid}&wforce=1&nogroup=false&fnick=&refer_lflag=&refer_from=profile_headerv6&_t=0`)
    }

    let scheduled = setInterval(() => {
        index += 1
        // 获取随机不重复的索引
        const randomIndex = getRandomIndex(length, usedIndexes);
        usedIndexes.push(randomIndex);
        follow(list[randomIndex]); // 关注随机值

        // 如果所有索引都被使用，则清空已使用索引列表，以重新开始
        if (usedIndexes.length === length) {
            usedIndexes.length = 0;
        }
    }, 5000)

    setTimeout(() => {
        clearTimeout(scheduled)
    }, 5100 * length)
})([3027806214,5991116818,1847588883])
```
		.catch(error => console.error(`出错了 (${error.message})${list.slice(index).length ? `\n\n以下 ID 还未处理，请之后再试\n\n${JSON.stringify(list.slice(index))}` : ''}`))
		.then(() => failed.length && console.warn(`以下 ID 关注失败，请重试或手动关注\n\n${JSON.stringify(failed)}`))
})(/*上一步的结果*/)
```
