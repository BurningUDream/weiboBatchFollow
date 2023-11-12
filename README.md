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
// 创建一个全局的 Map，用于保存关注的人名单
const globalMap = new Map();

// 创建一个计数器，用于记录全局 Map 的变化次数
let changeCount = 0;

// 添加一个函数，用于检查全局 Map 是否超过三次不再变化，并输出 Map
function checkAndOutputMap() {
  if (changeCount >= 3) {
    // 输出全局 Map
    console.log('全局 Map:', globalMap);
    
    // 停止监听滚动事件，以防止重复输出
    window.removeEventListener('scroll', updateGlobalMap);
  }
}

// 监听页面的滚动事件
function updateGlobalMap() {
  // 找到具有 usercard 属性的 span 元素
  const elements = document.querySelectorAll('span[usercard]');
  
  // 遍历每个元素并提取其文本内容，并以 usercard 为键，内容为值，加入全局 Map
  elements.forEach(element => {
    const usercard = element.getAttribute('usercard');
    const content = element.innerText;
    
    // 检查是否已经存在于全局 Map，如果不存在则加入
    if (!globalMap.has(usercard)) {
      globalMap.set(usercard, content);
      
      // 增加变化计数器
      changeCount++;
      
      // 重新设置计时器
      clearTimeout(outputTimeout);
      outputTimeout = setTimeout(checkAndOutputMap, 1000);
    }
  });
}

// 初始化页面滚动事件监听
window.addEventListener('scroll', updateGlobalMap);

// 初始化全局 Map 检查定时器
let outputTimeout = setTimeout(checkAndOutputMap, 3000);

// 获取globalMap的所有键（usercard值）
const keys = Array.from(globalMap.keys());

// 输出所有键
console.log('全局 Map 的所有键:', keys);
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
