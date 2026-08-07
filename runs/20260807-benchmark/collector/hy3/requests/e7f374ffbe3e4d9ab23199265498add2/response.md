以下是完整的单文件电梯模拟器代码。所有 HTML、CSS 和 JavaScript 均已内联，无需任何外部依赖。

### 电梯模拟器代码 (`elevator.html`)

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>电梯模拟器</title>
<style>
    * {
        box-sizing: border-box;
    }
    body {
        font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        background: #f0f2f5;
        margin: 0;
        padding: 20px;
        display: flex;
        flex-direction: column;
        align-items: center;
        -webkit-font-smoothing: antialiased;
    }
    h1 {
        color: #333;
        margin-top: 0;
    }
    /* 控制面板 */
    .controls {
        background: #fff;
        padding: 15px 20px;
        border-radius: 8px;
        box-shadow: 0 2px 8px rgba(0,0,0,0.1);
        margin-bottom: 20px;
        display: flex;
        gap: 15px;
        align-items: center;
    }
    .controls label {
        font-size: 14px;
        color: #555;
        font-weight: 500;
    }
    .controls select, .controls input {
        padding: 8px 12px;
        font-size: 14px;
        border: 1px solid #ccc;
        border-radius: 4px;
        margin-left: 5px;
    }
    .controls button {
        background: #3498db;
        color: #fff;
        border: none;
        padding: 9px 16px;
        font-size: 14px;
        border-radius: 4px;
        cursor: pointer;
        transition: background 0.2s;
        font-weight: bold;
    }
    .controls button:hover {
        background: #2980b9;
    }
    
    /* 模拟建筑区域 */
    .building {
        display: flex;
        height: 360px;
        gap: 20px;
        background: #fff;
        padding: 20px;
        border-radius: 8px;
        box-shadow: 0 4px 12px rgba(0,0,0,0.1);
    }
    
    /* 楼层与等待区 */
    .floors {
        flex: 1;
        display: flex;
        flex-direction: column;
        border-right: 2px solid #eee;
    }
    .floor {
        height: 60px;
        display: flex;
        align-items: center;
        border-bottom: 1px dashed #e0e0e0;
    }
    .floor:last-child {
        border-bottom: none;
    }
    .floor-label {
        width: 40px;
        font-weight: bold;
        color: #666;
        text-align: center;
        font-size: 16px;
    }
    .waiting-area {
        display: flex;
        flex-wrap: wrap;
        gap: 6px;
        margin-left: 10px;
        min-height: 24px;
        align-content: center;
    }
    
    /* 电梯井与电梯 */
    .elevators {
        position: relative;
        width: 210px;
        background: #fafafa;
        border: 1px solid #ddd;
        border-radius: 4px;
    }
    .elevator-shaft {
        position: absolute;
        top: 0;
        bottom: 0;
        width: 70px;
        border-right: 1px solid #eee;
    }
    .elevator-shaft:last-child {
        border-right: none;
    }
    .elevator {
        position: absolute;
        width: 60px;
        height: 50px;
        left: 5px;
        background: #3498db;
        border-radius: 6px;
        transition: top 1s linear, background-color 0.3s;
        display: flex;
        justify-content: center;
        align-items: center;
        box-shadow: inset 0 0 5px rgba(0,0,0,0.2);
        z-index: 5;
    }
    .elevator.door-open {
        background: #2ecc71; /* 开门状态变绿 */
    }
    
    /* 乘客与提示框 */
    .passenger {
        width: 24px;
        height: 24px;
        background: #e74c3c;
        border-radius: 50%;
        position: relative;
        cursor: pointer;
        box-shadow: 0 2px 4px rgba(0,0,0,0.3);
        transition: transform 0.2s;
    }
    .passenger:hover {
        transform: scale(1.15);
    }
    .passenger::after {
        content: attr(data-tooltip);
        position: absolute;
        bottom: 130%;
        left: 50%;
        transform: translateX(-50%);
        background: rgba(0,0,0,0.8);
        color: #fff;
        padding: 4px 8px;
        border-radius: 4px;
        font-size: 12px;
        white-space: nowrap;
        opacity: 0;
        pointer-events: none;
        transition: opacity 0.2s;
        z-index: 10;
    }
    .passenger:hover::after {
        opacity: 1;
    }
</style>
</head>
<body>

<h1>电梯模拟器</h1>

<div class="controls">
    <label>楼层:
        <select id="floor-select">
            <option value="0">0 层</option>
            <option value="1">1 层</option>
            <option value="2">2 层</option>
            <option value="3">3 层</option>
            <option value="4">4 层</option>
            <option value="5">5 层</option>
        </select>
    </label>
    <label>人数:
        <input type="number" id="num-people" value="1" min="1" max="20">
    </label>
    <button id="generate-btn">生成乘客</button>
</div>

<div class="building">
    <div class="floors" id="floors-container">
        <!-- 楼层将由 JS 动态生成 (从上到下: 5层 到 0层) -->
    </div>
    <div class="elevators" id="elevators-container">
        <!-- 电梯井与电梯将由 JS 动态生成 -->
    </div>
</div>

<script>
    const NUM_FLOORS = 6;     // 楼层数 0~5
    const NUM_ELEVATORS = 3;  // 电梯数量
    const FLOOR_HEIGHT = 60;  // 每层物理高度(px)

    let floors = Array.from({ length: NUM_FLOORS }, () => []);
    let elevators = [];
    let passengerIdCounter = 0;

    const floorsContainer = document.getElementById('floors-container');
    const elevatorsContainer = document.getElementById('elevators-container');

    // 1. 初始化楼层 DOM (从顶部5层到底部0层)
    for (let f = NUM_FLOORS - 1; f >= 0; f--) {
        const floorDiv = document.createElement('div');
        floorDiv.className = 'floor';
        floorDiv.innerHTML = `<div class="floor-label">${f}</div><div class="waiting-area" id="waiting-${f}"></div>`;
        floorsContainer.appendChild(floorDiv);
    }

    // 2. 初始化电梯井与电梯 DOM
    for (let i = 0; i < NUM_ELEVATORS; i++) {
        const shaft = document.createElement('div');
        shaft.className = 'elevator-shaft';
        shaft.style.left = (i * 70) + 'px';
        elevatorsContainer.appendChild(shaft);
    }
    for (let i = 0; i < NUM_ELEVATORS; i++) {
        const elevDiv = document.createElement('div');
        elevDiv.className = 'elevator';
        elevDiv.style.left = (i * 70 + 5) + 'px'; // 井道内居中
        elevDiv.style.top = getTopForFloor(0) + 'px'; // 初始均在 0 层
        elevatorsContainer.appendChild(elevDiv);

        elevators.push({
            id: i,
            element: elevDiv,
            currentFloor: 0,
            targetFloor: null,
            state: 'idle', // idle: 空闲, moving: 移动中, doorOpen: 开门状态
            passenger: null
        });
    }

    // 根据楼层计算电梯的 Y 轴坐标
    function getTopForFloor(floor) {
        return (NUM_FLOORS - 1 - floor) * FLOOR_HEIGHT;
    }

    // 3. 生成乘客逻辑
    document.getElementById('generate-btn').addEventListener('click', () => {
        const floor = parseInt(document.getElementById('floor-select').value);
        const num = parseInt(document.getElementById('num-people').value) || 1;
        const waitingArea = document.getElementById(`waiting-${floor}`);

        for (let i = 0; i < num; i++) {
            let target = Math.floor(Math.random() * NUM_FLOORS);
            while (target === floor) { // 确保目标楼层与当前楼层不同
                target = Math.floor(Math.random() * NUM_FLOORS);
            }

            const p = {
                id: passengerIdCounter++,
                targetFloor: target,
                element: document.createElement('div')
            };
            p.element.className = 'passenger';
            p.element.setAttribute('data-tooltip', `目标: ${target}层`);
            waitingArea.appendChild(p.element);
            floors[floor].push(p); // 加入该楼层的等待队列
        }
    });

    // 4. 电梯移动函数 (处理平滑动画和到达回调)
    function moveTo(elevator, floor, callback) {
        elevator.targetFloor = floor;
        elevator.state = 'moving';
        let distance = Math.abs(floor - elevator.currentFloor);
        let duration = Math.max(distance * 1.0, 0.1); // 每层移动耗时1秒
        
        elevator.element.style.transition = `top ${duration}s linear, background-color 0.3s`;
        elevator.element.style.top = getTopForFloor(floor) + 'px';

        setTimeout(() => {
            elevator.currentFloor = floor;
            callback();
        }, duration * 1000);
    }

    // 5. 调度算法：为空闲电梯寻找最近的且未被其他电梯认领的目标楼层
    function findTargetFloor(elevator) {
        let bestFloor = null;
        let minDistance = Infinity;

        for (let f = 0; f < NUM_FLOORS; f++) {
            if (floors[f].length > 0) {
                // 如果该楼层已有其他电梯正在前往(认领)，则不再重复派单
                let claimed = elevators.some(e => e !== elevator && e.targetFloor === f && e.state !== 'idle');
                if (!claimed) {
                    let distance = Math.abs(f - elevator.currentFloor);
                    if (distance < minDistance) {
                        minDistance = distance;
                        bestFloor = f;
                    }
                }
            }
        }
        return bestFloor;
    }

    // 6. 主调度循环 (每 200ms 检查一次状态)
    function tick() {
        elevators.forEach(elevator => {
            if (elevator.state === 'idle') {
                let target = findTargetFloor(elevator);
                if (target !== null) {
                    // 前往目标楼层接人
                    moveTo(elevator, target, () => {
                        elevator.state = 'doorOpen';
                        elevator.element.classList.add('door-open'); // 视觉开门
                        
                        setTimeout(() => {
                            elevator.element.classList.remove('door-open');
                            let queue = floors[elevator.currentFloor];
                            
                            // 每部电梯限乘1人，只接走队列最前方的1名乘客
                            if (queue.length > 0) {
                                let p = queue.shift();
                                elevator.element.appendChild(p.element); // 乘客进入电梯内部
                                elevator.passenger = p;
                                
                                // 接人后立刻关门，前往该乘客的目标楼层送人
                                moveTo(elevator, p.targetFloor, () => {
                                    elevator.state = 'doorOpen';
                                    elevator.element.classList.add('door-open');
                                    
                                    setTimeout(() => {
                                        elevator.element.classList.remove('door-open');
                                        p.element.remove(); // 乘客到达目的地，离开
                                        elevator.passenger = null;
                                        elevator.state = 'idle'; // 电梯重新变为空闲
                                    }, 500);
                                });
                            } else {
                                elevator.state = 'idle';
                            }
                        }, 600); // 到达后开门停留时间
                    });
                }
            }
        });
    }

    setInterval(tick, 200);

</script>
</body>
</html>
```

---

### 运行方式
1. 将以上完整代码复制并保存为 `elevator.html` 文件（确保文件后缀为 `.html`）。
2. 使用任意现代浏览器（如 Chrome、Edge、Firefox、Safari）直接双击打开该文件即可运行，无需搭建本地服务器或安装任何依赖。

---

### 测试说明
1. **生成乘客**：在页面左上角的控制面板中选择“楼层”与“人数”，点击“生成乘客”按钮。你会在对应楼层的等候区看到生成的红色圆点（乘客）。
2. **查看目标**：将鼠标悬停在任意红色乘客圆点上，即可弹出一个小提示框，显示该乘客随机分配的目标楼层。
3. **电梯调度与接送**：
   - 系统包含 3 部电梯（蓝色方块），初始状态均位于 0 层。
   - 电梯会自动调度前往有乘客等待的楼层。当电梯到达目标楼层时会变为绿色（表示开门），并接走 **1 名** 乘客（乘客进入电梯内居中显示）。
   - 由于每部电梯一次只能搭载 1 人，如果某层等待人数大于 1，当前电梯接走 1 人去往目的地后，剩余乘客将**留在原地**；下一部空闲电梯（或该电梯送完返回后）会再次响应并接走队列中的下一人，实现“依次上电梯”。
4. **平滑动画**：电梯上下移动具有与距离成正比的线性平滑过渡动画；乘客进入/离开电梯、电梯开门变绿均有流畅的视觉反馈。整体界面保持了简洁现代的设计风格。