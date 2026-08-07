用户想要一个电梯模拟器，用HTML/CSS/JavaScript实现。让我分析需求：

1. 多楼层建筑（用户可以在不同楼层生成人数）
2. 3部电梯
3. 每部电梯一次只能搭载一个人
4. 乘客上电梯后，其他乘客等待；当下一部电梯到达时，他们再依次上电梯
5. 每个人有随机目标楼层
6. 鼠标悬停显示目标楼层提示框
7. 流畅动画、良好视觉效果、简洁
8. 单个HTML文件

让我设计这个模拟器：

**建筑结构：**
- 假设有6层楼（从1到6）
- 每层楼有一个"生成乘客"按钮
- 显示等待乘客队列

**电梯系统：**
- 3部电梯，位置在建筑中
- 电梯一次只能载1人
- 电梯移动到有等待乘客的楼层，接载乘客，送到目标楼层

**乘客逻辑：**
- 乘客有起始楼层（生成楼层）和目标楼层（随机）
- 乘客在起始楼层等待
- 电梯到达后，一个乘客上电梯（一次一个）
- 电梯将乘客送到目标楼层后，乘客离开

**视觉设计：**
- 建筑垂直布局
- 电梯左右排列
- 乘客显示为小图标
- 悬停显示提示框

让我思考电梯调度算法：

最简单的方式：每个空闲电梯去响应最近的等待乘客请求。当电梯到达楼层且有乘客等待时，加载一个乘客，送往目标楼层，然后继续。

让我重新思考"一部电梯一次只能搭载一个人"的含义：电梯容量为1。所以电梯要么载着1个人，要么空载。

调度逻辑：
1. 电梯空闲时，寻找最近的等待乘客
2. 电梯到达乘客楼层，乘客上电梯
3. 电梯将乘客送到目标楼层
4. 乘客离开，电梯变为空闲

当多个人在相同楼层等待，电梯一次只能载一个人，所以需要多次往返。

让我设计更具体的逻辑：

**状态管理：**
- 建筑有多层，每层有等待队列（乘客数组）
- 3部电梯，每部有状态：空闲、移动接客、移动送客
- 每个乘客有：id、起始楼层、目标楼层、状态（等待、在电梯中、已到达）

**电梯行为：**
- 空闲时，寻找需要服务的请求（最近的等待乘客）
- 到达起始楼层后，让一个乘客上电梯
- 然后去目标楼层
- 到达后，乘客离开

我需要让动画流畅。使用requestAnimationFrame。

**数据结构：**

```javascript
const config = {
    floors: 6,
    elevators: 3,
    elevatorCapacity: 1
};

let passengers = [];
let elevators = [];
let waitingQueue = {}; // floor -> [passengerIds]

class Elevator {
    constructor(id) {
        this.id = id;
        this.currentFloor = 1;
        this.targetFloor = null;
        this.state = 'idle'; // idle, movingToPickup, movingToDropoff
        this.passenger = null;
        this.x = ...;
        this.y = ...;
    }
}
```

**具体实现步骤：**

1. HTML结构：建筑容器，楼层显示，电梯轨道，生成按钮
2. CSS样式：美观的电梯，楼层，乘客图标
3. JavaScript逻辑：乘客生成、电梯调度、动画更新

让我想想如何布局：

- 建筑从左到右分为：楼层区域（显示楼层号和生成按钮）、电梯区域（3部电梯）
- 每层楼高度约80-100px
- 电梯在这个区域内上下移动

为了简化，假设：
- 楼层从下到上：1楼在底部，6楼在顶部
- 垂直方向，floor y坐标 = 总高度 - floor * 楼层高度

让我用CSS+JS计算位置。

电梯轿厢的移动：使用absolute定位，通过transform translateY移动，或直接设置top。

让我用relative容器+absolute子元素。

**乘客视觉：**
- 乘客是小圆形或小人图标
- 在楼层等待区排队显示
- 悬停显示提示框（tooltip）

**调度算法细节：**

当电梯空闲时：
1. 找到所有有等待乘客的楼层
2. 选择最近的楼层（按电梯当前楼层与目标楼层的距离）
3. 移动去接乘客

当电梯到达接客楼层：
1. 从该楼层等待队列中取出一个乘客
2. 乘客进入电梯
3. 电梯设置目标为乘客的目标楼层
4. 移动送客

到达目标楼层：
1. 乘客离开电梯（标记为已到达）
2. 电梯变为空闲
3. 继续寻找下一个请求

这是一个简单的往返过程。由于电梯容量为1，每次只能服务一个乘客。

但要注意：乘客上电梯后，其他乘客留在原楼层等待。当电梯再次到达时，再上。

**生成乘客：**
- 每层有一个"+"按钮或"生成乘客"按钮
- 点击生成一个乘客，随机分配目标楼层（不等于当前楼层）
- 可能的补充：可以设置生成数量？需求说"允许用户在不同楼层生成一定数量的人"。所以需要一个输入或按钮。

让我提供每层一个按钮，点击生成1个乘客；或者可以输入数量。

简单起见：每层一个"生成乘客"按钮，点击生成1个随机目标楼层的乘客。也可以有一个输入框让用户输入数量。

需求说"允许用户在不同楼层生成一定数量的人"，所以最好有数量控制。让我简单做：每层有一个输入框和一个按钮，输入数量点击生成。

或者更简洁：每层一个按钮，点击一次生成一个。用户可以多次点击。

让我采用：每层有一个"生成乘客"按钮，点击生成1人。同时可能增加一个全局"随机生成"按钮，产生随机数量的乘客。

嗯，为了简洁，每层一个按钮即可。

**随机目标楼层：** 从1~6中随机选择一个不等于当前楼层的楼层。

**提示框：** 悬停在乘客上时，显示包含目标楼层的提示框。

让我设计CSS颜色的乘客：
- 等待的乘客：蓝色小圆
- 在电梯中的乘客：绿色小圆（或者电梯中的乘客显示在电梯内）

**布局详细设计：**

```
+------------------------------------------+
|  建筑模拟器                               |
+--------+-------------------------------+
| 楼层 6 |  [按钮]  [等待区]  | 电梯1 | 电梯2 | 电梯3 |
| 楼层 5 |  [按钮]  [等待区]  |       |       |       |
| 楼层 4 |  [按钮]  [等待区]  |       |       |       |
| 楼层 3 |  [按钮]  [等待区]  |       |       |       |
| 楼层 2 |  [按钮]  [等待区]  |       |       |       |
| 楼层 1 |  [按钮]  [等待区]  |       |       |       |
+--------+-------------------------------+
```

让我调整：电梯在楼层显示区域的右侧，每层楼有对应的位置。

建筑高度：假设6层，每层100px，建筑高600px，加上底部的一些空间。

让我用JS动态计算位置。

**电梯移动：**
电梯位置由currentFloor决定，通过CSS transition实现平滑移动。
```css
.elevator {
    transition: transform 0.5s ease-in-out;
}
```

或者用JS动画。

让我用requestAnimationFrame控制电梯移动速度，这样更可控。

**实现细节：**

电梯移动速度：比如每秒移动2层楼。用requestAnimationFrame更新位置。

每个电梯对象：
```javascript
{
    id: 0,
    currentFloor: 1,
    position: 1, // float，用于动画插值
    state: 'idle',
    passenger: null,
    targetFloor: null,
    // 视觉元素
    element: null
}
```

更新循环：
```javascript
function update(deltaTime) {
    elevators.forEach(elevator => {
        if (elevator.state === 'moving') {
            // 向目标楼层移动
            const speed = 2; // 层/秒
            const direction = Math.sign(elevator.targetFloor - elevator.currentFloor);
            elevator.position += direction * speed * deltaTime;
            
            // 到达
            if (Math.abs(elevator.position - elevator.targetFloor) < 0.05) {
                elevator.position = elevator.targetFloor;
                elevator.currentFloor = elevator.targetFloor;
                onElevatorArrive(elevator);
            }
            
            // 更新视觉位置
            updateElevatorVisual(elevator);
        }
    });
}
```

但这个逻辑需要更紧密的状态机集成。

让我重新设计状态机：

电梯状态：
- `idle`：空闲，停在某个楼层
- `movingPickup`：前往乘客所在楼层
- `movingDropoff`：送乘客到目标楼层

事件：
- `idle` → `movingPickup`：找到等待乘客
- 到达接客楼层：乘客上电梯，状态变为`movingDropoff`
- 到达目标楼层：乘客离开，状态变为`idle`

让我定义电梯的移动：

```javascript
class Elevator {
    constructor(id) {
        this.id = id;
        this.currentFloor = 1;
        this.targetFloor = null;
        this.state = 'idle';
        this.passenger = null;
        this.element = null;
    }
}
```

主循环：

```javascript
function update(deltaTime) {
    elevators.forEach(elevator => {
        if (elevator.state === 'idle') {
            // 寻找最近的等待乘客
            const request = findNearestRequest(elevator);
            if (request) {
                elevator.targetFloor = request.floor;
                elevator.state = 'movingPickup';
            }
        } else if (elevator.state === 'movingPickup' || elevator.state === 'movingDropoff') {
            // 移动
            const direction = Math.sign(elevator.targetFloor - elevator.currentFloor);
            elevator.currentFloor += direction * ELEVATOR_SPEED * deltaTime;
            
            // 检查是否到达
            if (Math.abs(elevator.currentFloor - elevator.targetFloor) < 0.05) {
                elevator.currentFloor = elevator.targetFloor;
                if (elevator.state === 'movingPickup') {
                    // 乘客上电梯
                    const passengerId = waitingQueue[elevator.targetFloor].shift();
                    const passenger = passengers.find(p => p.id === passengerId);
                    elevator.passenger = passenger;
                    passenger.state = 'inElevator';
                    elevator.state = 'movingDropoff';
                    elevator.targetFloor = passenger.targetFloor;
                } else if (elevator.state === 'movingDropoff') {
                    // 乘客到达
                    elevator.passenger.state = 'arrived';
                    elevator.passenger = null;
                    elevator.state = 'idle';
                }
            }
        }
    });
}
```

注意：`currentFloor`是浮点数，用于平滑移动。`Math.floor`或四舍五入用于显示。

但是当电梯到达接客楼层时，`targetFloor`是整数，所以应该能精确到达。

让我用position表示电梯连续位置，currentFloor表示四舍五入后的整数楼层。

让我简化：直接用`position`浮点数表示电梯在垂直方向上的位置。目标也是浮点数（楼层对应的y坐标）。

更好的方式：存储电梯的`y`坐标，目标y坐标。移动时更新。使用楼层数的概念只是为了逻辑。

让我定义：

- 楼层高度：FLOOR_HEIGHT = 100px
- 楼层i的y坐标：buildingHeight - (i - 0.5) * FLOOR_HEIGHT

算了，让我直接用CSS定位。电梯的`top`属性根据楼层计算。

**让我简化实现，采用如下方案：**

使用requestAnimationFrame驱动动画。电梯对象有currentFloor（浮点数，1~6）。每个frame根据速度移动。到达目标后处理事件。

**视觉设计：**

- 背景：浅灰色，建筑有现代感
- 楼层：清晰的层线，左侧显示楼层号
- 电梯：白色矩形，有边框，显示在里面的人（如果有）
- 乘客：彩色小圆，不同的颜色，悬停显示提示框

让我写代码吧。

**乘客生成：**

```javascript
function generatePassenger(floor) {
    const targetFloor = Math.floor(Math.random() * 6) + 1;
    while (targetFloor === floor) {
        targetFloor = Math.floor(Math.random() * 6) + 1;
    }
    const passenger = {
        id: nextPassengerId++,
        currentFloor: floor,
        targetFloor: targetFloor,
        state: 'waiting', // waiting, inElevator, arrived
        element: null
    };
    passengers.push(passenger);
    waitingQueue[floor].push(passenger.id);
    renderPassenger(passenger);
}
```

**渲染乘客：**

乘客在楼层等待区显示。每个等待乘客是一个小圆点，水平排列。

**悬停提示：**

使用CSS Tooltip或自定义tooltip。让我用CSS伪元素或JS控制的tooltip。

由于是纯HTML/CSS/JS，让我用简单的title属性？但title属性样式不统一。让我用自定义tooltip。

自定义tooltip：mouseover时显示div，mouseout时隐藏。

**电梯内饰：**

电梯内显示乘客（如果有）。乘客可以作为电梯内的一个小圆点。

**动画平滑：**

电梯移动用transition或JS动画。让我用JS动画配合requestAnimationFrame。

**让我重新考虑方案：**

为了代码简洁清晰，我将使用`requestAnimationFrame`驱动整个模拟器。每帧更新电梯位置，处理逻辑。

**详细计划：**

1. HTML结构：
```html
<div id="simulator">
  <div id="building">
    <div class="floor" data-floor="6">
      <div class="floor-label">6</div>
      <div class="waiting-area"></div>
      <button class="generate-btn">+</button>
    </div>
    ...
  </div>
  <div id="elevator-area">
    <div class="elevator-shaft">
      <div class="elevator" id="elevator-0"></div>
    </div>
    ...
  </div>
</div>
```

让我考虑更好的布局。电梯应该在楼层旁边。

其实，让我把楼层和电梯放在同一行。每层楼的布局：

```
[楼层号] [等待区] [电梯1] [电梯2] [电梯3]
```

电梯是在竖井里移动的，所以电梯应该在独立的层中，覆盖所有楼层。

更好的布局：

```
左侧：楼层信息区（楼层号、生成按钮、等待乘客）
右侧：电梯区域（3个电梯轨道，每个轨道内有电梯轿厢）
```

建筑总高度固定，电梯轿厢在轨道内上下移动。

**最终布局：**

```html
<div class="building">
  <!-- 楼层背景 -->
  <div class="floors-bg">
    <div class="floor-line" style="bottom: 0%">1F</div>
    <div class="floor-line" style="bottom: 20%">2F</div>
    ...
  </div>
  
  <!-- 乘客等待区 -->
  <div class="waiting-areas">
    <div class="waiting-area" data-floor="1"></div>
    ...
  </div>
  
  <!-- 电梯轨道 -->
  <div class="elevator-shafts">
    <div class="shaft">
      <div class="elevator" id="elevator-0"></div>
    </div>
    ...
  </div>
</div>
```

这样做比较复杂。让我简化，使用相对定位和绝对定位。

让我计算坐标：
- 建筑高度：600px（6层，每层100px）
- 1楼底部在600px，1楼中心在550px
- 6楼顶部在0px，6楼中心在50px

楼层中心y = 600 - (floor - 0.5) * 100 = 600 - floor*100 + 50 = 650 - floor*100

电梯也在这个坐标体系中定位。

让我们设定：

```javascript
const FLOORS = 6;
const FLOOR_HEIGHT = 100;
const BUILDING_HEIGHT = FLOORS * FLOOR_HEIGHT;
const ELEVATOR_SPEED = 2; // 楼层/秒

function getFloorY(floor) {
    return BUILDING_HEIGHT - (floor - 0.5) * FLOOR_HEIGHT;
}
```

电梯的`top`值 = getFloorY(elevator.currentFloor) - ELEVATOR_HEIGHT/2

**完整的实现代码：**

让我一次写完整。

我需要考虑：
1. 当多个电梯空闲时，如何分配最近的请求？
   - 每个空闲电梯独立寻找最近的等待乘客。
   - 这可能导致多个电梯去同一个楼层。为了避免，如果某个电梯已经决定去某个楼层，其他电梯就不应该再去。
   - 让我在findNearestRequest中排除已经被其他电梯选中的楼层。

2. 当楼层有多人等待，电梯每次只能载一人，所以电梯会多次往返。

3. 新生成的乘客应该立即出现在楼层等待区。

让我设计数据结构和逻辑：

```javascript
// 全局状态
const state = {
    elevators: [],
    passengers: [],
    waitingQueues: {}, // floor -> [passengerId]
    nextPassengerId: 1,
    lastTime: 0,
};

// 初始化
function init() {
    // 创建电梯
    for (let i = 0; i < 3; i++) {
        const elevator = {
            id: i,
            currentFloor: 1,
            targetFloor: null,
            state: 'idle', // 'idle', 'movingPickup', 'movingDropoff'
            passenger: null,
            element: document.getElementById(`elevator-${i}`)
        };
        state.elevators.push(elevator);
    }
    
    // 初始化等待队列
    for (let f = 1; f <= FLOORS; f++) {
        state.waitingQueues[f] = [];
    }
    
    // 开始动画循环
    requestAnimationFrame(update);
}
```

**电梯调度逻辑：**

```javascript
function findNearestRequest(elevator) {
    let bestFloor = null;
    let bestDistance = Infinity;
    
    for (let floor = 1; floor <= FLOORS; floor++) {
        if (state.waitingQueues[floor].length === 0) continue;
        
        // 检查是否已有电梯前往该楼层
        const alreadyServed = state.elevators.some(e => 
            e.id !== elevator.id && 
            (e.state === 'movingPickup' && e.targetFloor === floor)
        );
        if (alreadyServed) continue;
        
        const distance = Math.abs(elevator.currentFloor - floor);
        if (distance < bestDistance) {
            bestDistance = distance;
            bestFloor = floor;
        }
    }
    
    return bestFloor ? { floor: bestFloor, distance: bestDistance } : null;
}
```

注意：当电梯送客经过有等待乘客的楼层时，我们是否要停？需求中没明确。为简单起见，电梯只响应明确的请求，不在中途停靠。

**乘客上电梯：**

```javascript
function handleElevatorArrivePickup(elevator) {
    const floor = Math.round(elevator.currentFloor);
    const queue = state.waitingQueues[floor];
    if (queue.length === 0) {
        // 没有乘客了，电梯闲置
        elevator.state = 'idle';
        elevator.targetFloor = null;
        return;
    }
    
    const passengerId = queue.shift();
    const passenger = state.passengers.find(p => p.id === passengerId);
    if (!passenger) return;
    
    passenger.state = 'inElevator';
    elevator.passenger = passenger;
    elevator.state = 'movingDropoff';
    elevator.targetFloor = passenger.targetFloor;
    
    // 更新视觉
    passenger.element.remove();
    // 在电梯内显示乘客
    showPassengerInElevator(elevator, passenger);
}
```

**到达目标楼层：**

```javascript
function handleElevatorArriveDropoff(elevator) {
    const passenger = elevator.passenger;
    if (!passenger) return;
    
    passenger.state = 'arrived';
    // 移除电梯内的乘客显示
    passenger.element.remove();
    elevator.passenger = null;
    elevator.state = 'idle';
    elevator.targetFloor = null;
}
```

**动画循环：**

```javascript
function update(time) {
    const deltaTime = Math.min((time - state.lastTime) / 1000, 0.1);
    state.lastTime = time;
    
    state.elevators.forEach(elevator => {
        if (elevator.state === 'movingPickup' || elevator.state === 'movingDropoff') {
            const direction = Math.sign(elevator.targetFloor - elevator.currentFloor);
            elevator.currentFloor += direction * ELEVATOR_SPEED * deltaTime;
            
            // 检查是否到达目标楼层
            if ((direction > 0 && elevator.currentFloor >= elevator.targetFloor) ||
                (direction < 0 && elevator.currentFloor <= elevator.targetFloor) ||
                Math.abs(elevator.currentFloor - elevator.targetFloor) < 0.05) {
                elevator.currentFloor = elevator.targetFloor;
                
                if (elevator.state === 'movingPickup') {
                    handleElevatorArrivePickup(elevator);
                } else if (elevator.state === 'movingDropoff') {
                    handleElevatorArriveDropoff(elevator);
                }
            }
            
            // 更新电梯位置
            updateElevatorVisual(elevator);
        }
    });
    
    requestAnimationFrame(update);
}
```

**处理空闲电梯：**

在每次update中，对于空闲电梯，尝试寻找请求：

```javascript
state.elevators.forEach(elevator => {
    if (elevator.state === 'idle') {
        const request = findNearestRequest(elevator);
        if (request) {
            elevator.targetFloor = request.floor;
            elevator.state = 'movingPickup';
        }
    }
});
```

**渲染乘客：**

每个乘客是一个div，绝对定位在楼层等待区。

```javascript
function createPassengerElement(passenger) {
    const el = document.createElement('div');
    el.className = 'passenger';
    el.dataset.passengerId = passenger.id;
    
    // 随机颜色（基于id）
    const colors = ['#ff6b6b', '#4ecdc4', '#45b7d1', '#f9ca24', '#e056fd', '#6ab04c'];
    el.style.backgroundColor = colors[passenger.id % colors.length];
    
    // 提示框
    const tooltip = document.createElement('div');
    tooltip.className = 'tooltip';
    tooltip.textContent = `目标: ${passenger.targetFloor}F`;
    el.appendChild(tooltip);
    
    // 悬停事件
    el.addEventListener('mouseenter', () => tooltip.style.display = 'block');
    el.addEventListener('mouseleave', () => tooltip.style.display = 'none');
    
    return el;
}
```

**等待区布局：**

每个楼层的等待区是一个水平区域，乘客从左到右排列。如果人多，可以换行。

**电梯内的乘客显示：**

电梯内部空间有限，在电梯内显示一个小圆点。

**让我也考虑楼层生成按钮：**

每个楼层左侧有一个"+ 生成"按钮，点击生成一个乘客。

**视觉设计细节：**

- 建筑背景：深色渐变
- 楼层：有横线分隔
- 电梯：白色轿厢，有玻璃感
- 等待乘客：彩色小圆，有shadow
- 电梯内的乘客：在电梯中显示

**工具提示：**

需求提到"鼠标悬停在人物上时，显示一个包含其目标楼层的小提示框"。所以要对每个乘客实现悬停提示。

**让我开始写完整的代码：**

我需要在单个HTML文件中完成所有内容。

让我重新思考坐标体系：

建筑容器：宽800px，高660px（6层*100 + 底部60px的缓冲区）
但为了让最底层（1楼）在底部，通常建筑底部应该接近容器底部。

让我设定：
- 建筑容器高度：620px (6层 * 100 + 20px padding)
- 楼层高度：100px
- 1楼底部：600px，1楼中心：550px
- 6楼顶部：0px，6楼中心：50px

所以 `getFloorY(floor) = 650 - floor * 100`

电梯轿厢高度：60px，宽度：60px

**整体HTML结构：**

```html
<div id="app">
  <h1>🏢 电梯模拟器</h1>
  <div id="building">
    <div id="floors">
      <!-- 楼层标签和生成按钮 -->
    </div>
    <div id="waiting-area">
      <!-- 等待乘客 -->
    </div>
    <div id="elevators">
      <!-- 3部电梯 -->
    </div>
  </div>
</div>
```

让我用更精确的CSS定位：

```html
<div id="building">
  <!-- 楼层背景网格 -->
  <div class="floor-lines">
    <div class="floor-line" style="bottom: 100px">5F</div>
    ...
  </div>
  
  <!-- 等待区域 -->
  <div class="floor-row" data-floor="6">
    <span class="floor-num">6</span>
    <button class="gen-btn">+</button>
    <div class="waiting-area"></div>
  </div>
  ...
</div>
```

这样每一行代表一个楼层，电梯在其中移动。但是电梯需要跨越楼层移动，所以不能放在行内。

让我分层设计：
1. `floors-container` - 包含6层的网格背景和等待区
2. `elevators-container` - 绝对定位覆盖在整个建筑上，包含3个电梯轨道和电梯

电梯轨道是垂直的，每个轨道是一个列，从建筑顶部到底部。

让我们这样布局：

```html
<div id="building">
  <div id="floors-panel">
    <!-- 每层一行 -->
    <div class="floor-row">
      <span class="floor-num">6F</span>
      <button>生成</button>
      <div class="waiting-area"></div>
    </div>
    ...
  </div>
  
  <div id="elevators-panel">
    <div class="elevator-shaft">
      <div class="elevator"></div>
    </div>
    <div class="elevator-shaft">
      <div class="elevator"></div>
    </div>
    <div class="elevator-shaft">
      <div class="elevator"></div>
    </div>
  </div>
</div>
```

`#floors-panel`是正常流布局，占据左侧宽度。
`#elevators-panel`是绝对定位，覆盖在右侧。

或者用flex布局，左侧楼层区，右侧电梯区。电梯区相对定位，电梯绝对定位在轨道内移动。

让我用flex布局：

```html
<div id="building">
  <div id="left-panel">
    <!-- 6层，每层显示楼层号、按钮、等待区 -->
  </div>
  <div id="right-panel">
    <div class="elevator-shaft" id="shaft-0">
      <div class="elevator" id="elevator-0"></div>
    </div>
    <div class="elevator-shaft" id="shaft-1">
      <div class="elevator" id="elevator-1"></div>
    </div>
    <div class="elevator-shaft" id="shaft-2">
      <div class="elevator" id="elevator-2"></div>
    </div>
  </div>
</div>
```

右侧面板高度为BUILDING_HEIGHT，每个轨道宽度80px，三个并排。
电梯在轨道内通过`top`属性定位。

轨道内电梯的top = getFloorY(floor) - ELEVATOR_HEIGHT/2

让我更仔细地计算：
- getFloorY(floor) = BUILDING_HEIGHT - (floor - 0.5) * FLOOR_HEIGHT
- 当floor=1时：600 - 50 = 550
- 当floor=6时：600 - 550 = 50

所以电梯的top = 550 - 30 = 520（1楼时）
6楼时：50 - 30 = 20

这样，电梯在轨道内上下移动，轨道从顶部0到底部600，但电梯不会超出范围。

**等待区的位置：**

左侧面板中，每层行的高度是100px。等待区在行内。

乘客元素在等待区内绝对定位或flex排列。

**关于乘客的创建和显示：**

```javascript
function createPassengerElement(passenger) {
    const div = document.createElement('div');
    div.className = 'passenger';
    div.style.background = `hsl(${passenger.id * 47 % 360}, 70%, 60%)`;
    
    const tooltip = document.createElement('span');
    tooltip.className = 'passenger-tooltip';
    tooltip.textContent = `目标: ${passenger.targetFloor}层`;
    div.appendChild(tooltip);
    
    div.addEventListener('mouseenter', () => {
        tooltip.style.display = 'block';
    });
    div.addEventListener('mouseleave', () => {
        tooltip.style.display = 'none';
    });
    
    return div;
}
```

**生成乘客：**

```javascript
function generatePassenger(floor) {
    let targetFloor;
    do {
        targetFloor = Math.floor(Math.random() * FLOORS) + 1;
    } while (targetFloor === floor);
    
    const passenger = {
        id: nextPassengerId++,
        currentFloor: floor,
        targetFloor: targetFloor,
        state: 'waiting',
        element: null
    };
    
    state.passengers.push(passenger);
    state.waitingQueues[floor].push(passenger.id);
    
    const el = createPassengerElement(passenger);
    passenger.element = el;
    document.querySelector(`.floor-row[data-floor="${floor}"] .waiting-area`).appendChild(el);
    
    // 通知空闲电梯
    return passenger;
}
```

**电梯调度优化：**

当生成乘客后，如果有空闲电梯，应该立即响应。但我们已经在主循环中处理了空闲电梯的调度。只要主循环在运行，空闲电梯就会找到请求。

**处理电梯内部乘客显示：**

电梯内部有一个div用于显示乘客。当乘客上电梯后，在电梯内显示乘客的小圆点。

```html
<div class="elevator">
  <div class="elevator-passenger"></div>
</div>
```

**电梯运动的速度：**

让我用相对速度：2层/秒。这样从1楼到6楼需要2.5秒。挺合理的。

**代码实现：**

让我把全部逻辑写出来。

关于tooltip的位置：tooltip应该在乘客上方或下方显示。由于乘客在楼层等待区，空间有限，tooltip可以绝对定位在乘客上方。

但需要注意，tooltip可能会被截断。让我在CSS中处理。

**让我也考虑当乘客到达时，如何从视觉上移除：**

当乘客到达目标楼层，乘客会从电梯中出来并消失。可以有一个动画效果（淡出）。

**完整的实现：**

OK，让我写代码。

**全局变量：**

```javascript
const FLOORS = 6;
const FLOOR_HEIGHT = 100;
const BUILDING_HEIGHT = FLOORS * FLOOR_HEIGHT; // 600
const ELEVATOR_SPEED = 2; // 层/秒
const ELEVATOR_COUNT = 3;
const ELEVATOR_HEIGHT = 70;
const ELEVATOR_WIDTH = 60;

let nextPassengerId = 1;
let lastTime = 0;

const elevators = [];
const passengers = [];
const waitingQueues = {};
```

**初始化：**

```javascript
function init() {
    // 初始化等待队列
    for (let f = 1; f <= FLOORS; f++) {
        waitingQueues[f] = [];
    }
    
    // 创建电梯
    for (let i = 0; i < ELEVATOR_COUNT; i++) {
        const elevator = {
            id: i,
            currentFloor: 1,
            targetFloor: null,
            state: 'idle',
            passenger: null,
            element: document.getElementById(`elevator-${i}`),
            passengerElement: document.getElementById(`elevator-${i}-passenger`)
        };
        elevators.push(elevator);
        updateElevatorVisual(elevator);
    }
    
    // 开始动画
    requestAnimationFrame(loop);
}
```

**主循环：**

```javascript
function loop(time) {
    const deltaTime = Math.min((time - lastTime) / 1000, 0.1);
    lastTime = time;
    
    // 调度空闲电梯
    elevators.forEach(elevator => {
        if (elevator.state === 'idle') {
            const request = findNearestRequest(elevator);
            if (request) {
                elevator.targetFloor = request.floor;
                elevator.state = 'movingPickup';
            }
        }
    });
    
    // 移动电梯
    elevators.forEach(elevator => {
        if (elevator.state === 'movingPickup' || elevator.state === 'movingDropoff') {
            const direction = Math.sign(elevator.targetFloor - elevator.currentFloor);
            elevator.currentFloor += direction * ELEVATOR_SPEED * deltaTime;
            
            // 到达检查
            if ((direction > 0 && elevator.currentFloor >= elevator.targetFloor) ||
                (direction < 0 && elevator.currentFloor <= elevator.targetFloor)) {
                elevator.currentFloor = elevator.targetFloor;
                handleElevatorArrival(elevator);
            }
            
            updateElevatorVisual(elevator);
        }
    });
    
    requestAnimationFrame(loop);
}
```

**处理电梯到达：**

```javascript
function handleElevatorArrival(elevator) {
    if (elevator.state === 'movingPickup') {
        const floor = Math.round(elevator.currentFloor);
        const queue = waitingQueues[floor];
        
        if (queue.length === 0) {
            elevator.state = 'idle';
            elevator.targetFloor = null;
            return;
        }
        
        const passengerId = queue.shift();
        const passenger = passengers.find(p => p.id === passengerId);
        if (passenger) {
            passenger.state = 'inElevator';
            elevator.passenger = passenger;
            
            // 从等待区移除
            passenger.element.remove();
            
            // 在电梯中显示
            elevator.passengerElement.style.backgroundColor = passenger.color;
            elevator.passengerElement.style.display = 'block';
            
            elevator.state = 'movingDropoff';
            elevator.targetFloor = passenger.targetFloor;
        }
    } else if (elevator.state === 'movingDropoff') {
        const passenger = elevator.passenger;
        if (passenger) {
            passenger.state = 'arrived';
            
            // 从电梯中移除
            elevator.passengerElement.style.display = 'none';
            elevator.passenger = null;
        }
        
        elevator.state = 'idle';
        elevator.targetFloor = null;
    }
}
```

**寻找最近的请求：**

```javascript
function findNearestRequest(elevator) {
    let bestFloor = null;
    let bestDistance = Infinity;
    
    for (let floor = 1; floor <= FLOORS; floor++) {
        if (waitingQueues[floor].length === 0) continue;
        
        // 检查是否已被其他电梯响应
        const alreadyAssigned = elevators.some(e => 
            e.id !== elevator.id && 
            e.state === 'movingPickup' && 
            e.targetFloor === floor
        );
        if (alreadyAssigned) continue;
        
        const distance = Math.abs(elevator.currentFloor - floor);
        if (distance < bestDistance) {
            bestDistance = distance;
            bestFloor = floor;
        }
    }
    
    return bestFloor ? { floor: bestFloor } : null;
}
```

**更新电梯视觉：**

```javascript
function updateElevatorVisual(elevator) {
    const y = getFloorY(elevator.currentFloor) - ELEVATOR_HEIGHT / 2;
    elevator.element.style.top = `${y}px`;
}
```

**getFloorY：**

```javascript
function getFloorY(floor) {
    return BUILDING_HEIGHT - (floor - 0.5) * FLOOR_HEIGHT;
}
```

**创建乘客元素：**

```javascript
function createPassengerElement(passenger) {
    const div = document.createElement('div');
    div.className = 'passenger';
    div.style.backgroundColor = passenger.color;
    
    const tooltip = document.createElement('span');
    tooltip.className = 'passenger-tooltip';
    tooltip.textContent = `目标: ${passenger.targetFloor}F`;
    div.appendChild(tooltip);
    
    div.addEventListener('mouseenter', () => tooltip.style.display = 'block');
    div.addEventListener('mouseleave', () => tooltip.style.display = 'none');
    
    return div;
}
```

**生成乘客函数：**

```javascript
function generatePassenger(floor) {
    let targetFloor;
    do {
        targetFloor = Math.floor(Math.random() * FLOORS) + 1;
    } while (targetFloor === floor);
    
    const passenger = {
        id: nextPassengerId++,
        currentFloor: floor,
        targetFloor: targetFloor,
        state: 'waiting',
        color: `hsl(${Math.random() * 360}, 70%, 60%)`,
        element: null
    };
    
    passengers.push(passenger);
    waitingQueues[floor].push(passenger.id);
    
    const el = createPassengerElement(passenger);
    passenger.element = el;
    
    const waitingArea = document.querySelector(`.floor-row[data-floor="${floor}"] .waiting-area`);
    waitingArea.appendChild(el);
}
```

**HTML结构：**

让我组装HTML。

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<title>电梯模拟器</title>
<style>
  /* CSS */
</style>
</head>
<body>
<div id="app">
  <h1>🏢 电梯模拟器</h1>
  <div id="building">
    <div id="floors-panel">
      <!-- 动态生成楼层 -->
    </div>
    <div id="elevators-panel">
      <!-- 动态生成电梯 -->
    </div>
  </div>
</div>
<script>
  // JS
</script>
</body>
</html>
```

让我生成楼层的HTML。

**楼层面板：**

```html
<div id="floors-panel">
  <div class="floor-row" data-floor="6">
    <span class="floor-num">6F</span>
    <button class="gen-btn">+</button>
    <div class="waiting-area"></div>
  </div>
  ...
</div>
```

由于楼层是从上到下排列的，6楼在最上面。在循环中，我让floor从FLOORS开始递减。

**电梯面板：**

```html
<div id="elevators-panel">
  <div class="elevator-shaft">
    <div class="elevator" id="elevator-0">
      <div class="elevator-passenger" id="elevator-0-passenger"></div>
    </div>
  </div>
  ...
</div>
```

**CSS样式：**

让我设计美观的样式。

```css
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

body {
    font-family: 'Segoe UI', sans-serif;
    background: linear-gradient(135deg, #0c0c1d 0%, #1a1a2e 100%);
    min-height: 100vh;
    display: flex;
    justify-content: center;
    align-items: center;
}

#app {
    background: #1a1a2e;
    border-radius: 16px;
    padding: 20px;
    box-shadow: 0 8px 32px rgba(0,0,0,0.3);
}

h1 {
    color: #e0e0e0;
    text-align: center;
    margin-bottom: 15px;
    font-size: 24px;
}

#building {
    display: flex;
    gap: 0;
    background: #2d2d44;
    border-radius: 12px;
    overflow: hidden;
    position: relative;
    height: 620px;
}

#floors-panel {
    width: 400px;
    display: flex;
    flex-direction: column;
}

.floor-row {
    height: 100px;
    border-bottom: 1px solid #3a3a55;
    display: flex;
    align-items: center;
    padding: 0 15px;
    gap: 10px;
    position: relative;
    background: rgba(255,255,255,0.02);
}

.floor-row:last-child {
    border-bottom: none;
}

.floor-num {
    color: #aaa;
    font-weight: bold;
    font-size: 18px;
    width: 40px;
}

.gen-btn {
    background: #4ecdc4;
    border: none;
    border-radius: 50%;
    width: 32px;
    height: 32px;
    font-size: 18px;
    cursor: pointer;
    color: #1a1a2e;
    transition: all 0.2s;
    display: flex;
    align-items: center;
    justify-content: center;
}

.gen-btn:hover {
    background: #6ee0d8;
    transform: scale(1.1);
}

.waiting-area {
    flex: 1;
    display: flex;
    flex-wrap: wrap;
    align-content: center;
    gap: 6px;
    min-height: 40px;
    padding: 5px;
}

.passenger {
    width: 24px;
    height: 24px;
    border-radius: 50%;
    position: relative;
    cursor: pointer;
    transition: transform 0.2s;
    box-shadow: 0 2px 6px rgba(0,0,0,0.3);
    border: 2px solid rgba(255,255,255,0.3);
}

.passenger:hover {
    transform: scale(1.3);
    z-index: 10;
}

.passenger-tooltip {
    display: none;
    position: absolute;
    bottom: 30px;
    left: 50%;
    transform: translateX(-50%);
    background: #333;
    color: white;
    padding: 4px 10px;
    border-radius: 6px;
    font-size: 12px;
    white-space: nowrap;
    z-index: 100;
    box-shadow: 0 2px 8px rgba(0,0,0,0.4);
    pointer-events: none;
}

.passenger-tooltip::after {
    content: '';
    position: absolute;
    top: 100%;
    left: 50%;
    transform: translateX(-50%);
    border: 4px solid transparent;
    border-top-color: #333;
}

/* 电梯面板 */
#elevators-panel {
    display: flex;
    gap: 10px;
    padding: 10px;
    background: #22223a;
    border-left: 2px solid #333350;
    width: 260px;
    position: relative;
}

.elevator-shaft {
    flex: 1;
    position: relative;
    height: 600px;
    background: #1c1c30;
    border-radius: 8px;
    border: 1px solid #333350;
    overflow: hidden;
}

.elevator {
    position: absolute;
    width: 80%;
    height: 70px;
    background: linear-gradient(145deg, #4a4a6a, #3a3a55);
    border: 2px solid #5a5a7a;
    border-radius: 8px;
    left: 10%;
    display: flex;
    justify-content: center;
    align-items: center;
    box-shadow: 0 4px 12px rgba(0,0,0,0.4);
    transition: top 0.1s linear;
    z-index: 10;
}

.elevator-passenger {
    width: 20px;
    height: 20px;
    border-radius: 50%;
    display: none;
    border: 2px solid white;
    box-shadow: 0 2px 4px rgba(0,0,0,0.3);
}
```

等等，`transition: top 0.1s linear` 可能会与requestAnimationFrame冲突。让我去掉transition，直接用top定位。

由于我们使用requestAnimationFrame每帧更新位置，不需要CSS过渡。移除transition。

**完整代码需要仔细处理一些边界情况：**

1. 当电梯正在移动去接客的途中，如果等待的乘客已经被其他电梯接走（不太可能，因为我们有alreadyAssigned检查，但万一），应该在到达时重新检查。
2. 电梯空闲后，应该立即重新分配任务。这在主循环的开头处理。

让我也考虑在乘客到达后，让等待区闪烁一下或产生特效。

**让我也增加到达动画：**

当乘客到达目标楼层时，可以在楼层上短暂显示乘客到达的标记，然后消失。

不过为了简洁，可以让乘客淡出。

**工具提示的改进：**

提示框应该显示"目标楼层: X"，并且要确保在鼠标悬停时可见。由于乘客是动态生成和销毁的，要注意事件绑定。

**楼层背景的改进：**

为了更清晰，可以在每个楼层右侧显示楼层号（在电梯轨道旁边）。

让我在马桶里加一些细节：

```css
.floor-row::before {
    content: '';
    position: absolute;
    left: 0;
    right: 0;
    top: 0;
    height: 2px;
    background: #3a3a55;
}
```

**最终代码：**

让我把完整的代码写出来。这是一个比较大的文件，但所有内容都在一个HTML中。

我这里需要仔细处理一些东西：

1.  `Math.round(elevator.currentFloor)` 在到达检查中，我们使用浮点数移动，到达时 `currentFloor = targetFloor`，所以取整没问题。

2.  当电梯在同一楼层（例如，电梯在1楼，乘客在1楼），distance = 0，电梯应该立即接客。由于主循环中，我们检查 `elevator.state === 'movingPickup'` 并移动，但如果 currentFloor == targetFloor，direction = 0，不会进入移动分支。我应该特别处理这种情况。

让我在分配任务时检查：
```javascript
if (request.floor === elevator.currentFloor) {
    // 立即处理
    handleElevatorArrival(elevator); // 但state还没设为movingPickup？
}
```

更好的处理：在找到请求后，如果当前就在目标楼层，直接处理：

```javascript
if (request.floor === elevator.currentFloor) {
    elevator.state = 'movingPickup';
    elevator.targetFloor = request.floor;
    handleElevatorArrival(elevator);
} else {
    elevator.state = 'movingPickup';
    elevator.targetFloor = request.floor;
}
```

这样`handleElevatorArrival`会检查当前楼层，如果队列有乘客就接走。

嗯，但实际上在`handleElevatorArrival`中，我们期望state是`movingPickup`。所以我要确保。

但还有一个问题：如果电梯已经到达，`handleElevatorArrival`将乘客接走并设置state为`movingDropoff`，然后下一轮循环会继续处理移动。这没问题。

让我调整逻辑：

在分配任务时，可以直接：
```javascript
elevator.targetFloor = request.floor;
elevator.state = 'movingPickup';
handleElevatorArrival(elevator);
```

因为`handleElevatorArrival`检查`elevator.state === 'movingPickup'`，并从等待队列中接走乘客。但如果`elevator.currentFloor === elevator.targetFloor`，则方向为0，不会进入移动分支。所以立即调用是安全的。

但我要小心，如果`handleElevatorArrival`失败（队列为空），state会变为idle。

好的，让我实现这个逻辑。

3. 更新电梯视觉时，currentFloor可能不是整数。`getFloorY`接受浮点数，所以计算结果也是浮点数，这没问题。电梯会平滑移动。

4. 关于电梯门动画：为了简洁，我不做开门关门动画。

5. 乘客在等待区使用flex布局，左右排列。当乘客到达后，需要从DOM中移除。

让我现在写完整代码。

**HTML结构生成：**

我会在JS中动态生成楼层和电梯的DOM，而不是硬编码HTML。这样更灵活。

但为了让代码更清晰，让我用JS动态生成。

**完整代码：**

让我用内存来编写。

让我再考虑一些细节：

- 乘客的颜色：可以用HSL生成，保证多样性。
- 乘客大小：24px的圆。
- 电梯大小：宽60px，高70px。

现在让我评估一下整体布局：

建筑可能是：左侧楼层面板宽约400px，右侧电梯区域宽约260px。总宽约660px。高620px。

在较小的屏幕上可能不适应，但我们的目标是演示，可以接受。

让我调整宽度：左400 + 右280 = 680px。再加上padding等，约720px。

**关于"每部电梯一次只能搭载一个人"：**

这意味着电梯容量为1。所以电梯内最多显示一个乘客。

**关于"当下一部电梯到达时"：**

这意味着，如果多人在同一楼层等待，而一部电梯到达，只能上一个。其他人继续等待。当下一部电梯到达时（可能是同一部电梯返回，或其他电梯），再上一个。

**关于测试：**

我需要提供测试说明。让我在交付时给出简单的测试步骤。

好的，现在开始写完整代码。

让我重新设定变量名和结构，确保代码质量。

实际上，让我重新考虑电梯的调度算法。目前的算法是：每个空闲电梯独立寻找最近的请求。这可能导致一个问题：当多个电梯空闲时，它们可能会选择同一个楼层（因为第一个电梯找到请求后状态就不再是idle了，所以第二个电梯不会和它冲突）。让我们验证：

1. 电梯A空闲，寻找请求，找到楼层3有乘客，state变为movingPickup。
2. 电梯B空闲，寻找请求，遍历楼层，发现楼层3已有电梯A前往（alreadyAssigned），所以跳过，去别的楼层。

这很好。

但是，当电梯A到达楼层3，接走一个乘客后，楼层3可能还有等待乘客。此时电梯A去送乘客了。电梯B如果已经在去楼层3的路上（不太可能，因为B会避开已被分配的目标楼层），但如果在电梯A接走乘客后，B才查找，B会看到楼层3有等待乘客，于是也去楼层3。

这是合理的：不同电梯可以服务同一楼层的不同乘客。

**另一个问题：电梯送客经过有等待乘客的楼层时，是否应该停下来接客？**

需求中没说，为了简单，不停。只服务明确的目标。

**让我完善代码：**

现在，我考虑到有些用户可能会快速点击生成多个乘客，楼层等待区可能会拥挤。等待区的flex-wrap可以解决这个问题。

让我设定每个乘客的tooltip样式，确保悬停时可见。

**关于乘客的静态显示：**

对于已经到达的乘客，为了简洁，直接移除DOM元素。

**关于整体视觉：**

让我使用深色主题，配合霓虹灯效果，营造现代感。

**代码组织：**

1. 常量定义
2. 全局状态
3. 初始化函数
4. 乘客管理（生成、创建DOM）
5. 电梯调度
6. 电梯移动和处理
7. 动画循环
8. 事件绑定

让我开始写吧。

让我再想想更具体的实现细节。

**关于楼层行的高度和建筑高度的匹配：**

左侧面板的楼层行高100px，6行=600px。右侧电梯轨道高度也应该是600px。但建筑面板还有标题，所以总高度要相应增加。

让我把建筑高度设为620px（包括顶部的一些空间）。实际上，左侧6行就是600px，没有额外空间。两侧都要对齐。

让我把右侧的电梯轨道高度设为600px，与左侧面板的600px对齐。

电梯轨道中的电梯，其top值根据getFloorY计算，范围从50到550，那么电梯（高70px）的top范围是从15到515，都在轨道范围内。

**让我也添加当前电梯位置的指示灯或显示：**

在电梯轨道旁边可以显示电梯当前所在楼层。

但为了简洁，可以让电梯本身显示楼层号，或者在轨道底部显示电梯编号。

**让我处理电梯的初始位置：**

所有电梯初始在1楼。

**关于CSS过渡：**

我可以同时使用CSS过渡和JS更新。但为了避免冲突，直接使用JS更新top值，不使用CSS过渡。

由于每帧更新，动画会很平滑。

**代码中可能的Bug注意点：**

1. 当电梯到达乘客楼层时，如果乘客队列为空，需要处理。
2. 当多个电梯同时到达同一楼层，需要保证只服务一个乘客。但由于我们使用`shift()`从队列中取乘客，所以是原子的，不会冲突。

**另一个重要的调度改进：**

当前的算法中，电梯一旦空闲就寻找最近的请求。但如果乘客不断生成，电梯可能一直在工作。这是合理的。

让我也处理这种情况：所有电梯都忙，但又有新乘客。新乘客会等待，直到有电梯空闲。

**好的，让我写最终代码。我需要非常仔细。**

让我也加入电梯的状态图标：
- 空闲：绿色指示灯
- 移动中：黄色指示灯
- 即将到达：无特殊指示

这会让视觉效果更好。

**让我用以下结构来实现：**

电梯元素内：
- 指示灯（左上角小圆）
- 乘客（如果有）
- 编号（底部）

**最终实现：**

我会注意把所有CSS写清楚，确保视觉美观。

让我开始码代码吧。这将是很大的一个文件，我先组织好。

---

实际上，让我重新考虑一下。我需要确保代码能够在单个HTML文件中运行，并且不需要外部依赖。使用原生HTML/CSS/JS。

**让我设计最终UI：**

```
+----------------------------------------------------------+
|           🏢 电梯模拟器                                    |
+------------+---------------------------------------------+
|            |  电梯1    │  电梯2    │  电梯3    │          |
| 6F  [+]   |  [  ]    │  [  ]    │  [  ]    │          |
|            |          │          │          │          |
| 5F  [+]   |  [  ]    │  [  ]    │  [  ]    │          |
|            |          │          │          │          |
| 4F  [+]   |  [  ]    │  [  ]    │  [  ]    │          |
|            |          │          │          │          |
| 3F  [+]   |  [  ]    │  [  ]    │  [  ]    │          |
|            |          │          │          │          |
| 2F  [+]   |  [  ]    │  [  ]    │  [  ]    │          |
|            |          │          │          │          |
| 1F  [+]   |  [  ]    │  [  ]    │  [  ]    │          |
+------------+---------------------------------------------+
```

左侧每一行包含：楼层号、生成按钮、等待乘客区域。
右侧是三个电梯轨道。

**关于等待区的布局：**

每层的等待区在一行内，最多显示若干乘客。如果超出，滚动或换行。

我使用flex-wrap: wrap来换行。

**现在开始代码：**

我会写一个完整的HTML文件。

让我特别注意：
- tooltip要正确显示目标楼层。
- 乘客到达后要给出视觉反馈（可能是一瞬间的动画然后消失）。

**到达动画：**

乘客到达后，可以在目标楼层的等待区旁边显示一个短暂的"+1"效果，然后淡出。

不过为了简洁，我跳过这个动画，只是让乘客直接消失。

**电梯移动的精确性问题：**

电梯的currentFloor是浮点数。当它接近目标楼层时，使用`Math.abs(elevator.currentFloor - elevator.targetFloor) < 0.05`作为到达条件会更安全。但我的移动逻辑是检查方向，所以：

```javascript
if ((direction > 0 && elevator.currentFloor >= elevator.targetFloor) ||
    (direction < 0 && elevator.currentFloor <= elevator.targetFloor)) {
    elevator.currentFloor = elevator.targetFloor;
    handleElevatorArrival(elevator);
}
```

这个逻辑是可靠的。当direction=0时（在同一楼层），不会进入移动和处理。所以需要单独处理。

在分配任务时：
```javascript
if (Math.abs(elevator.currentFloor - request.floor) < 0.01) {
    elevator.state = 'movingPickup';
    elevator.targetFloor = request.floor;
    handleElevatorArrival(elevator);
} else {
    elevator.state = 'movingPickup';
    elevator.targetFloor = request.floor;
}
```

不过，`handleElevatorArrival`中需要处理state为'movingPickup'的情况。这样安排是正确的。

让我用更统一的方式：在分配任务时，设置state，然后立即尝试处理到达（如果已经在目标楼层）。如果处理成功，state会变为其他状态；如果不能处理（队列为空），state会变为idle。

```javascript
elevator.targetFloor = request.floor;
elevator.state = 'movingPickup';
handleElevatorArrival(elevator);
```

在`handleElevatorArrival`中：
```javascript
function handleElevatorArrival(elevator) {
    if (elevator.state === 'movingPickup') {
        const floor = Math.round(elevator.currentFloor);
        const queue = waitingQueues[floor];
        if (queue.length === 0) {
            elevator.state = 'idle';
            elevator.targetFloor = null;
            return;
        }
        // ... 接乘客
    } else if (elevator.state === 'movingDropoff') {
        // ... 放下乘客
    }
}
```

由于在`handleElevatorArrival`中设置了新的state和targetFloor，所以后续的调度逻辑会正确响应。

但是，当`handleElevatorArrival`从`movingPickup`变为`movingDropoff`时，如果新的targetFloor等于当前楼层（不太可能，因为目标楼层不能等于起始楼层），需要立即处理？不需要，因为目标楼层和当前楼层不同（生成乘客时保证）。

好的，这个逻辑没问题。

**让我也考虑：当两部电梯都去同一个楼层接客人时（但只有一部被分配），另一部不会去。但如果有两个乘客在同一楼层，而两部电梯都空闲，第二部电梯会看到该楼层有等待乘客，并且目标楼层未被分配？让我们检查：**

- 电梯A空闲，找到楼层3有2个等待乘客。设置targetFloor=3，state='movingPickup'。
- 电梯B空闲，找到楼层3有2个等待乘客。检查`alreadyAssigned`，发现电梯A已经前往3楼，所以跳过。
- 电梯B去找其他楼层。

这很好。但电梯A到达3楼后，接走1个乘客，还剩1个。电梯A离开后，楼层3仍有1个等待乘客。
- 之后电梯B空闲，会看到楼层3有等待乘客，于是去3楼接。

这是合理的。

**关于多乘客同时生成的问题：**

如果用户点击快速生成多个乘客，每个乘客有随机目标楼层。等待区的DOM可能同时添加多个乘客元素。这没问题。

**让我们也考虑：乘客从目标楼层出来后的状态：**

为了简单，乘客到达后直接消失，不显示在楼层中。

好的，让我现在写完整的代码。

我需要考虑文件长度。一个完整的实现大概400-600行。

让我开始写。

哦等等，我还需要考虑每层生成乘客的"一定数量"。用户可能需要输入数量。为了简化，我提供：
- 每层一个"+ 生成1人"按钮，点击一次生成1人。
- 也可以有一个输入框，输入数量，点击生成。

为了简洁，我用按钮+输入框的方式：每层一个输入框和一个生成按钮。输入框默认1。

但这样UI稍复杂。让我用两种方式结合：点击按钮生成1个，长按或连续点击生成多个。

为了最简洁，我每层只放一个"+"按钮，点击生成1个。用户多次点击可以生成多个人。

这样最简单。

让我再考虑增加一个"批量生成"功能：每层有一个输入框和按钮。

算了，就一个按钮吧。需求是"允许用户在不同楼层生成一定数量的人"，点击生成即可。

**让我也考虑在楼层行中加入输入框：**

```html
<div class="floor-row">
  <span class="floor-num">6F</span>
  <input type="number" min="1" max="10" value="1" style="width: 45px">
  <button>生成</button>
  <div class="waiting-area"></div>
</div>
```

这样更灵活。但输入框会让界面略复杂。让我采用：按钮+"生成人数"输入框。

嗯，我还是用简单的方式：一个按钮，点击一次生成1个乘客。

**但需求原文是"允许用户在不同楼层生成一定数量的人"——可以理解为一个按钮点击就生成一个，数量由点击次数决定。**

好，就一个按钮。

让我用更直观的方式：每层一个"+"按钮，旁边显示该层等待人数。

这也是一种可视化。

**开始写吧：**

让我整理下HTML结构：

```html
<div id="app">
  <h1>🏢 电梯模拟器</h1>
  <div id="building">
    <div id="floors-panel" class="panel">
      <!-- JS动态生成 -->
    </div>
    <div id="elevators-panel" class="panel">
      <!-- JS动态生成 -->
    </div>
  </div>
</div>
```

让我用JS生成楼层和电梯。

**完整实现：**

让我用JavaScript直接动态创建DOM，减少HTML硬编码。

我将在`init()`中创建所有DOM。

**这里有一个问题：** 动态创建DOM后，需要维护对DOM元素的引用，以便更新。我的设计是：

- `floorsData`数组：每个元素包含floor编号、等待区DOM引用。
- `elevators`数组：每个元素包含电梯DOM引用。

让我构建。

好的，我决定完整地实现它。下面是我将编写的代码的框架：

```javascript
// 常量
const FLOORS = 6;
const FLOOR_HEIGHT = 100;
const BUILDING_HEIGHT = FLOORS * FLOOR_HEIGHT;
const ELEVATOR_SPEED = 2;
const ELEVATOR_COUNT = 3;
const ELEVATOR_HEIGHT = 70;

// 状态
const state = {
    elevators: [],
    passengers: [],
    waitingQueues: {},
    nextPassengerId: 1,
    lastTime: 0,
};

// 初始化DOM
function initDOM() {
    const floorsPanel = document.getElementById('floors-panel');
    const elevatorsPanel = document.getElementById('elevators-panel');
    
    // 创建楼层
    for (let f = FLOORS; f >= 1; f--) {
        const row = document.createElement('div');
        row.className = 'floor-row';
        row.dataset.floor = f;
        
        const label = document.createElement('span');
        label.className = 'floor-num';
        label.textContent = `${f}F`;
        
        const btn = document.createElement('button');
        btn.className = 'gen-btn';
        btn.textContent = '+';
        btn.title = `在${f}层生成乘客`;
        btn.addEventListener('click', () => generatePassenger(f));
        
        const waitingArea = document.createElement('div');
        waitingArea.className = 'waiting-area';
        
        row.appendChild(label);
        row.appendChild(btn);
        row.appendChild(waitingArea);
        floorsPanel.appendChild(row);
    }
    
    // 创建电梯
    for (let i = 0; i < ELEVATOR_COUNT; i++) {
        const shaft = document.createElement('div');
        shaft.className = 'elevator-shaft';
        
        const elevatorEl = document.createElement('div');
        elevatorEl.className = 'elevator';
        elevatorEl.id = `elevator-${i}`;
        
        const passengerEl = document.createElement('div');
        passengerEl.className = 'elevator-passenger';
        passengerEl.id = `elevator-${i}-passenger`;
        
        const labelEl = document.createElement('div');
        labelEl.className = 'elevator-label';
        labelEl.textContent = `${i + 1}`;
        
        elevatorEl.appendChild(passengerEl);
        elevatorEl.appendChild(labelEl);
        shaft.appendChild(elevatorEl);
        elevatorsPanel.appendChild(shaft);
    }
}

// 初始化电梯
function initElevators() {
    for (let i = 0; i < ELEVATOR_COUNT; i++) {
        state.elevators.push({
            id: i,
            currentFloor: 1,
            targetFloor: null,
            state: 'idle',
            passenger: null,
            element: document.getElementById(`elevator-${i}`),
            passengerElement: document.getElementById(`elevator-${i}-passenger`),
        });
    }
    updateAllElevators();
}

// 更新电梯视觉位置
function updateElevatorVisual(elevator) {
    const y = getFloorY(elevator.currentFloor) - ELEVATOR_HEIGHT / 2;
    elevator.element.style.top = `${y}px`;
}

function updateAllElevators() {
    state.elevators.forEach(updateElevatorVisual);
}

// 获取楼层Y坐标
function getFloorY(floor) {
    return BUILDING_HEIGHT - (floor - 0.5) * FLOOR_HEIGHT;
}

// 创建乘客DOM
function createPassengerDOM(passenger) {
    const el = document.createElement('div');
    el.className = 'passenger';
    el.style.backgroundColor = passenger.color;
    
    const tooltip = document.createElement('span');
    tooltip.className = 'passenger-tooltip';
    tooltip.textContent = `目标: ${passenger.targetFloor}F`;
    el.appendChild(tooltip);
    
    el.addEventListener('mouseenter', () => tooltip.style.display = 'block');
    el.addEventListener('mouseleave', () => tooltip.style.display = 'none');
    
    return el;
}

// 生成乘客
function generatePassenger(floor) {
    let targetFloor;
    do {
        targetFloor = Math.floor(Math.random() * FLOORS) + 1;
    } while (targetFloor === floor);
    
    const passenger = {
        id: state.nextPassengerId++,
        currentFloor: floor,
        targetFloor,
        state: 'waiting',
        color: `hsl(${Math.random() * 360}, 70%, 60%)`,
        element: null,
    };
    
    state.passengers.push(passenger);
    state.waitingQueues[floor].push(passenger.id);
    
    const el = createPassengerDOM(passenger);
    passenger.element = el;
    
    const row = document.querySelector(`.floor-row[data-floor="${floor}"]`);
    const waitingArea = row.querySelector('.waiting-area');
    waitingArea.appendChild(el);
    
    // 更新该层等待人数显示
    updateFloorStatus(floor);
}

// 更新楼层状态显示
function updateFloorStatus(floor) {
    const row = document.querySelector(`.floor-row[data-floor="${floor}"]`);
    const count = state.waitingQueues[floor].length;
    // 可以更新一个计数器元素
}
```

**主循环和调度：**

```javascript
function loop(time) {
    const deltaTime = Math.min((time - state.lastTime) / 1000, 0.1);
    state.lastTime = time;
    
    // 调度空闲电梯
    state.elevators.forEach(elevator => {
        if (elevator.state === 'idle') {
            const request = findNearestRequest(elevator);
            if (request) {
                elevator.targetFloor = request.floor;
                elevator.state = 'movingPickup';
                
                // 如果已经在目标楼层，立即到达
                if (Math.abs(elevator.currentFloor - elevator.targetFloor) < 0.01) {
                    handleElevatorArrival(elevator);
                }
            }
        }
    });
    
    // 移动电梯
    state.elevators.forEach(elevator => {
        if (elevator.state === 'movingPickup' || elevator.state === 'movingDropoff') {
            const direction = Math.sign(elevator.targetFloor - elevator.currentFloor);
            elevator.currentFloor += direction * ELEVATOR_SPEED * deltaTime;
            
            if ((direction > 0 && elevator.currentFloor >= elevator.targetFloor) ||
                (direction < 0 && elevator.currentFloor <= elevator.targetFloor)) {
                elevator.currentFloor = elevator.targetFloor;
                handleElevatorArrival(elevator);
            }
            
            updateElevatorVisual(elevator);
        }
    });
    
    requestAnimationFrame(loop);
}
```

**处理到达：**

```javascript
function handleElevatorArrival(elevator) {
    if (elevator.state === 'movingPickup') {
        const floor = Math.round(elevator.currentFloor);
        const queue = state.waitingQueues[floor];
        
        if (queue.length === 0) {
            elevator.state = 'idle';
            elevator.targetFloor = null;
            return;
        }
        
        const passengerId = queue.shift();
        const passenger = state.passengers.find(p => p.id === passengerId);
        if (!passenger) {
            elevator.state = 'idle';
            elevator.targetFloor = null;
            return;
        }
        
        passenger.state = 'inElevator';
        elevator.passenger = passenger;
        
        // 从等待区移除
        passenger.element.remove();
        
        // 在电梯中显示
        elevator.passengerElement.style.backgroundColor = passenger.color;
        elevator.passengerElement.style.display = 'block';
        
        elevator.state = 'movingDropoff';
        elevator.targetFloor = passenger.targetFloor;
        
        // 更新等待区状态
        updateFloorStatus(floor);
        
    } else if (elevator.state === 'movingDropoff') {
        const passenger = elevator.passenger;
        if (passenger) {
            passenger.state = 'arrived';
            passenger.element.remove(); // 如果有残留DOM
        }
        
        // 清空电梯乘客显示
        elevator.passengerElement.style.display = 'none';
        elevator.passengerElement.style.backgroundColor = 'transparent';
        elevator.passenger = null;
        
        elevator.state = 'idle';
        elevator.targetFloor = null;
    }
}
```

**找到最近请求：**

```javascript
function findNearestRequest(elevator) {
    let bestFloor = null;
    let bestDistance = Infinity;
    
    for (let floor = 1; floor <= FLOORS; floor++) {
        if (state.waitingQueues[floor].length === 0) continue;
        
        const alreadyAssigned = state.elevators.some(e =>
            e.id !== elevator.id &&
            e.state === 'movingPickup' &&
            e.targetFloor === floor
        );
        if (alreadyAssigned) continue;
        
        const distance = Math.abs(elevator.currentFloor - floor);
        if (distance < bestDistance) {
            bestDistance = distance;
            bestFloor = floor;
        }
    }
    
    return bestFloor ? { floor: bestFloor } : null;
}
```

**CSS设计：**

让我现在设计美观的CSS。

首先，整体深色主题，搭配霓虹灯元素。

```css
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

body {
    font-family: 'Segoe UI', Tahoma, sans-serif;
    background: linear-gradient(135deg, #0b0e14 0%, #1a1c28 100%);
    min-height: 100vh;
    display: flex;
    justify-content: center;
    align-items: center;
}

#app {
    background: linear-gradient(145deg, #20222e, #292c3a);
    border-radius: 20px;
    padding: 25px 30px;
    box-shadow: 0 20px 60px rgba(0,0,0,0.5);
    border: 1px solid #333850;
}

h1 {
    color: #e8e8f0;
    text-align: center;
    margin-bottom: 20px;
    font-size: 26px;
    letter-spacing: 2px;
    text-shadow: 0 2px 8px rgba(78, 205, 196, 0.3);
}

#building {
    display: flex;
    background: #1a1c28;
    border-radius: 12px;
    overflow: hidden;
    border: 1px solid #2a2d3d;
}

#floors-panel {
    width: 420px;
    border-right: 2px solid #2a2d3d;
}

.floor-row {
    height: 100px;
    display: flex;
    align-items: center;
    padding: 0 15px;
    gap: 10px;
    border-bottom: 1px solid #242836;
    position: relative;
    transition: background 0.3s;
}

.floor-row:hover {
    background: rgba(255,255,255,0.04);
}

.floor-row:last-child {
    border-bottom: none;
}

.floor-num {
    color: #6a6d7a;
    font-weight: 700;
    font-size: 18px;
    width: 45px;
    text-shadow: 0 1px 2px rgba(0,0,0,0.5);
}

.gen-btn {
    background: linear-gradient(145deg, #4ecdc4, #3bb0a8);
    border: none;
    border-radius: 50%;
    width: 36px;
    height: 36px;
    font-size: 20px;
    font-weight: bold;
    color: #0b0e14;
    cursor: pointer;
    transition: all 0.25s;
    box-shadow: 0 4px 12px rgba(78, 205, 196, 0.3);
    display: flex;
    align-items: center;
    justify-content: center;
}

.gen-btn:hover {
    transform: scale(1.1);
    box-shadow: 0 6px 16px rgba(78, 205, 196, 0.5);
}

.waiting-area {
    flex: 1;
    display: flex;
    flex-wrap: wrap;
    align-items: center;
    gap: 8px;
    min-height: 40px;
    padding: 8px;
    background: rgba(0,0,0,0.2);
    border-radius: 8px;
}

.passenger {
    width: 26px;
    height: 26px;
    border-radius: 50%;
    position: relative;
    cursor: pointer;
    transition: transform 0.2s, box-shadow 0.2s;
    box-shadow: 0 3px 8px rgba(0,0,0,0.4);
    border: 2px solid rgba(255,255,255,0.4);
    animation: popIn 0.3s ease-out;
}

.passenger:hover {
    transform: scale(1.4);
    box-shadow: 0 4px 14px rgba(0,0,0,0.6);
    z-index: 20;
}

@keyframes popIn {
    from { transform: scale(0); opacity: 0; }
    to { transform: scale(1); opacity: 1; }
}

.passenger-tooltip {
    display: none;
    position: absolute;
    bottom: 32px;
    left: 50%;
    transform: translateX(-50%);
    background: rgba(20, 22, 30, 0.95);
    color: #fff;
    padding: 6px 12px;
    border-radius: 8px;
    font-size: 12px;
    white-space: nowrap;
    z-index: 100;
    box-shadow: 0 4px 16px rgba(0,0,0,0.4);
    pointer-events: none;
    border: 1px solid #333850;
}

.passenger-tooltip::after {
    content: '';
    position: absolute;
    top: 100%;
    left: 50%;
    transform: translateX(-50%);
    border: 6px solid transparent;
    border-top-color: rgba(20, 22, 30, 0.95);
}

/* 电梯区域 */
#elevators-panel {
    display: flex;
    gap: 15px;
    padding: 15px;
    background: #1a1c28;
    width: 280px;
}

.elevator-shaft {
    flex: 1;
    position: relative;
    height: 600px;
    background: linear-gradient(180deg, #0e1018, #141620);
    border-radius: 10px;
    border: 1px solid #2a2d3d;
    overflow: hidden;
    box-shadow: inset 0 2px 12px rgba(0,0,0,0.4);
}

.elevator-shaft::before {
    content: '';
    position: absolute;
    top: 0;
    bottom: 0;
    left: 50%;
    width: 2px;
    background: rgba(255,255,255,0.03);
}

.elevator {
    position: absolute;
    width: 85%;
    height: 70px;
    background: linear-gradient(145deg, #3d4a5d, #2d3645);
    border: 2px solid #4a5a6d;
    border-radius: 10px;
    left: 7.5%;
    display: flex;
    justify-content: center;
    align-items: center;
    box-shadow: 0 6px 16px rgba(0,0,0,0.5), inset 0 1px 2px rgba(255,255,255,0.1);
    z-index: 10;
}

.elevator-passenger {
    width: 20px;
    height: 20px;
    border-radius: 50%;
    display: none;
    border: 2px solid white;
    box-shadow: 0 2px 6px rgba(0,0,0,0.4);
}

.elevator-label {
    position: absolute;
    bottom: 4px;
    right: 6px;
    font-size: 10px;
    color: rgba(255,255,255,0.4);
}

.elevator-idle {
    box-shadow: 0 0 8px rgba(78, 205, 196, 0.2), 0 6px 16px rgba(0,0,0,0.5);
}

.elevator-busy {
    box-shadow: 0 0 12px rgba(240, 200, 80, 0.3), 0 6px 16px rgba(0,0,0,0.5);
}
```

等等，因为电梯状态变化，我需要添加/删除class来反映状态。

让我在状态变化时更新电梯样式：

```javascript
function updateElevatorStyle(elevator) {
    elevator.element.classList.remove('elevator-idle', 'elevator-busy');
    if (elevator.state === 'idle') {
        elevator.element.classList.add('elevator-idle');
    } else {
        elevator.element.classList.add('elevator-busy');
    }
}
```

在handleElevatorArrival中调用。

**关于电梯门的视觉：**

我可以在电梯元素中加一条缝线来表示门。

为了简洁，不做了。

**等待区的人数指示器：**

在楼层行的右侧，可以显示一个等待人数的数字。

```html
<div class="waiting-count">3人等待</div>
```

这个也可以作为视觉反馈。

让我加上。

好的，让我最终把代码写完整。

让我确保所有DOM操作都有正确的引用。

在`generatePassenger`中，我需要找到对应楼层的waiting-area元素。可以在初始化时保存引用：

```javascript
const floorElements = {};
// 在创建楼层时：
floorElements[f] = {
    row, waitingArea
};
```

这样更高效。

让我用这种方式。

**关于电梯调度的再思考：**

当前算法中，电梯仅在有等待乘客时才响应。但如果乘客已经在电梯里，电梯正在送客途中，它的状态是'movingDropoff'，不会被调度。

当电梯到达目标楼层，乘客离开后，电梯变为idle，然后在下一帧循环中，它会找到新的请求。

这有一个小延迟（一帧），但几乎感觉不到。

**关于动画效果：**

乘客上电梯后，电梯内显示乘客的小圆点。乘客的颜色与等待时一致。这有助于追踪。

**关于工具提示：**

工具提示在乘客悬停时显示。包含目标楼层信息。

**让我把最终的完整代码写出来。**

我需要特别注意代码的完整性和可运行性。

**让我再考虑一个细节：** 当乘客到达目标楼层后，是否应该显示一个到达通知？

简单起见，不显示。直接消失。

**但需求中说"鼠标悬停在人物上时，显示一个包含其目标楼层的小提示框"。** 这个"人物"包括等待中的人和电梯中的人吗？

"一名乘客上电梯后，其他乘客应留在原地等待" - 这表明有乘客在电梯中，也有乘客在等待。悬停提示应该对所有乘客都有效。但电梯乘客在电梯内，鼠标悬停在电梯上时，应显示乘客的目标楼层。

为了简洁，我让电梯内的乘客也有tooltip。但这些乘客元素在电梯内，可以悬停。

让我把tooltip机制也应用到电梯内的乘客。

但电梯内的乘客是一个小的div，悬停空间有限。让我在电梯元素的悬停上显示乘客信息。

一种做法：在电梯元素上绑定mouseenter/mouseleave，显示乘客的tooltip。

```javascript
// 当乘客上电梯后
elevator.element.title = `目标: ${passenger.targetFloor}F`;
```

但title的样式不美观。让我用自定义tooltip。

好吧，为了简洁，我可以在电梯的乘客元素上使用tooltip，就像等待区一样。

由于电梯内的乘客元素是`elevator-passenger`，我可以给它加上tooltip：

```javascript
const tooltip = document.createElement('span');
tooltip.className = 'passenger-tooltip';
tooltip.textContent = `目标: ${passenger.targetFloor}F`;
elevator.passengerElement.appendChild(tooltip);
elevator.passengerElement.addEventListener('mouseenter', () => tooltip.style.display = 'block');
elevator.passengerElement.addEventListener('mouseleave', () => tooltip.style.display = 'none');
```

但`elevator-passenger`是一个div，当没有乘客时display:none。当有乘客时显示。tooltip会在其中。

不过，电梯内的乘客元素显示为小圆，hover区域太小。为了更好的体验，可以在整个电梯元素上悬停时显示乘客目标。

那么我可以在电梯元素上绑定事件：

```javascript
// 在乘客进入电梯时：
elevator.element.setAttribute('data-tooltip', `目标: ${passenger.targetFloor}F`);
```

然后通过CSS或JS显示tooltip。

让我实现：在电梯元素上绑定mouseenter/mouseleave，如果电梯有乘客，显示一个tooltip。

但这样会与乘客的tooltip重复。简单起见，让我直接在电梯元素中渲染tooltip，并放在显眼位置。

让我用一种更简洁的方式：电梯乘客的小圆也有tooltip，而且电梯本身可以有一个"正在前往X楼"的小标签。

算了，让我不过度设计。只对等待区乘客做tooltip，因为需求描述是"鼠标悬停在人物上"，人物应该包括在电梯里的乘客，但为了简洁，电梯里的乘客也可以用同样的机制。

我的`elevator-passenger`元素可以复用`createPassengerDOM`的逻辑，但需要不同的样式。

让我这样解决：在`handleElevatorArrival`中，为电梯乘客创建tooltip。

好的，让我在代码中实现。

现在让我整体写一遍代码。这是一个大文件，但我会仔细组织。

等一下，让我重新审视需求的一句话："一名乘客上电梯后，其他乘客应留在原地等待；当下一部电梯（三部中的任意一部）到达时，他们再依次上电梯。"

这意味着：每次电梯到达一个楼层，只能带走一个乘客。其他乘客等待下一部电梯。这是符合我们的实现的。

"每个人都有一个随机分配的目标楼层。" - 这已经实现。

"鼠标悬停在人物上时，显示一个包含其目标楼层的小提示框。" - 这已经实现。

"整体应具有流畅的动画、良好的视觉效果，并保持简洁。" - 我们将实现。

好的，开始最终代码。

让我也添加一个统计面板：显示总等待人数、总到达人数。但为了简洁，我可以不添加。

**最终决定：**

为了简洁，不添加太多额外UI。核心功能齐全。

---

现在让我编写代码。这需要仔细斟酌，确保没有bug。

让我把所有代码写在一个HTML文件中。

让我在脑海里模拟一下运行测试：

1. 打开页面，6层楼，3部电梯都在1楼。
2. 点击5楼的"+"生成一个乘客，目标楼层随机（假设2楼）。
3. 电梯1（当前1楼）空闲，找到最近请求：5楼。移动去5楼。
4. 电梯1到达5楼，接走乘客，乘客进入电梯。电梯1目标变为2楼。
5. 电梯1到达2楼，乘客离开。电梯1空闲。
6. 如果此时电梯2和电梯3也在1楼空闲，它们也会竞相寻找请求。但电梯1已经接走乘客，所以没有请求了。

让我们测试多乘客：

1. 在5楼生成2个乘客：A（目标2楼），B（目标4楼）。
2. 电梯1和2都在1楼。电梯1先调度，去5楼（距离4）。电梯2调度，发现5楼已有电梯1前往，跳过。电梯2没有其他请求，保持空闲。
3. 电梯1到5楼，接走A（目标2楼），电梯1离开。
4. 电梯2此时空闲，发现5楼还有B等待，去5楼。
5. 电梯1到2楼放A，回到idle。
6. 电梯2到5楼接B，去4楼放B。

这看起来合理。

测试3部电梯同时接客：

1. 在6楼、5楼、4楼各生成一个乘客。
2. 三部电梯都在1楼。
3. 电梯1最近的是6楼（距离5），电梯2去5楼（距离4），电梯3去4楼（距离3）。等等，距离相同？1楼到4楼是3，到5楼是4，到6楼是5。所以电梯3去4楼，电梯2去5楼，电梯1去6楼。

但注意：findNearestRequest返回最近的，对于电梯1，最近的是4楼（距离3），所以电梯1去4楼。电梯2去5楼，电梯3去6楼。这样分配是合理的。

很好。

好的，我准备开始写代码了。

在代码中，我需要确保：
- floor-row的data-floor属性与楼层号匹配。
- 等待区的DOM操作正确。

让我考虑更细腻的UI设计：

**楼层行：**
- 左侧楼层号
- 中间等待区（乘客圆点在此排队）
- 右侧生成按钮

或者：
- 左侧楼层号 + "生成"按钮
- 中间等待区

为了不挤占等待区空间，我把生成按钮放在楼层号旁边。

**让我们也显示电梯当前状态：**

在电梯轨道下方或者旁边，可以显示电梯的当前楼层和状态。但为了简洁，可以在电梯元素上显示当前楼层号。

让我在电梯元素内显示一个小数字，表示当前楼层。

```html
<div class="elevator">
  <span class="elevator-floor-num">1F</span>
  <div class="elevator-passenger"></div>
</div>
```

这样每次更新位置时更新这个数字。

**好，现在写代码。我会很仔细。**

让我把所有逻辑汇总，写成一个HTML文件。

这里需要注意一行：`state.waitingQueues[floor].push(passenger.id)` 中的`floor`变量，我用了楼层号。在`findNearestRequest`中，也是在楼层1-6中遍历。这两个要一致。

初始化时：
```javascript
for (let f = 1; f <= FLOORS; f++) {
    state.waitingQueues[f] = [];
}
```

好。

**最终代码长度估算：**
- CSS 约250行
- HTML 约30行
- JS 约300行
总共约580行。

好的，让我开始。

**在写代码时的一个注意点：**

对于tooltip，由于乘客元素在鼠标进入/离开时显示/隐藏tooltip，需要注意mouseenter和mouseleave冒泡问题。使用`mouseenter/mouseleave`（不冒泡）是安全的。

**关于生成按钮的交互：**

用户点击生成按钮，生成一个乘客。为了满足"一定数量的人"，用户需要多次点击。可以在按钮上添加一个数字输入框，让用户输入生成数量。但为了简洁，我用一次点击生成一个人。

让我用稍微不同的方式：点击按钮后用`Math.random`生成1-3个乘客？不合理，需求允许"一定数量"。

还是就用点击一次一个人。用户多点几次就可以生成多个。

或者，我可以在生成按钮旁边加一个小数字输入框（1-5），每次点击生成对应数量。

嗯，让我加上输入框。看起来更符合"一定数量的人"。

```html
<div class="floor-row">
  <span class="floor-num">6F</span>
  <input type="number" min="1" max="5" value="1" class="gen-count">
  <button class="gen-btn">生成</button>
  <div class="waiting-area"></div>
</div>
```

这样用户可以设置生成数量。但会让界面有点拥挤。

让我换一种方式：每层一个"+"按钮，一个"-"按钮，用来增加/减少生成数量？这样更复杂。

让我采用最简洁的：每层一个按钮，点击生成1个乘客。同时全局有一个"批量生成"按钮，随机在任意层生成多个乘客？这会增加复杂性。

算了，就每层一个"+"按钮，点击生成1个乘客。多次点击生成多个。这是最简单的。

**但需求说"允许用户在不同楼层生成一定数量的人"**，这个"一定数量"是用户决定的。多次点击就是多次生成。

好的，就用一个"+"按钮。

让我在按钮上加上title提示："在6楼生成乘客"。

好的，最终决定。

让我在编写代码的时候加入一个计数器，在按钮旁边显示该层等待人数。这有助于用户了解各层状态。

```html
<span class="wait-count">3人</span>
```

好的。

**开始编写最终代码。**

让我先在脑海中构建完整的JS：

```javascript
const FLOORS = 6;
const FLOOR_HEIGHT = 100;
const BUILDING_HEIGHT = FLOORS * FLOOR_HEIGHT;
const ELEVATOR_SPEED = 2; // 层/秒
const ELEVATOR_COUNT = 3;
const ELEVATOR_HEIGHT = 70;

const state = {
    elevators: [],
    passengers: [],
    waitingQueues: {},
    nextPassengerId: 1,
    lastTime: 0,
};

function getFloorY(floor) {
    return BUILDING_HEIGHT - (floor - 0.5) * FLOOR_HEIGHT;
}

// ---------- DOM构建 ----------
function setupDOM() {
    const floorsPanel = document.getElementById('floors-panel');
    const elevatorsPanel = document.getElementById('elevators-panel');
    
    for (let f = FLOORS; f >= 1; f--) {
        const row = document.createElement('div');
        row.className = 'floor-row';
        row.dataset.floor = f;
        
        const label = document.createElement('span');
        label.className = 'floor-num';
        label.textContent = `${f}F`;
        
        const btn = document.createElement('button');
        btn.className = 'gen-btn';
        btn.textContent = '+';
        btn.title = `在${f}层生成乘客`;
        btn.addEventListener('click', () => generatePassenger(f));
        
        const count = document.createElement('span');
        count.className = 'wait-count';
        count.textContent = '0人';
        count.id = `wait-count-${f}`;
        
        const waitingArea = document.createElement('div');
        waitingArea.className = 'waiting-area';
        waitingArea.id = `waiting-area-${f}`;
        
        row.appendChild(label);
        row.appendChild(btn);
        row.appendChild(count);
        row.appendChild(waitingArea);
        floorsPanel.appendChild(row);
    }
    
    for (let i = 0; i < ELEVATOR_COUNT; i++) {
        const shaft = document.createElement('div');
        shaft.className = 'elevator-shaft';
        
        const elevatorEl = document.createElement('div');
        elevatorEl.className = 'elevator';
        elevatorEl.id = `elevator-${i}`;
        
        const passengerEl = document.createElement('div');
        passengerEl.className = 'elevator-passenger';
        passengerEl.id = `elevator-${i}-passenger`;
        
        const floorNum = document.createElement('span');
        floorNum.className = 'elevator-floor-num';
        floorNum.textContent = '1F';
        
        elevatorEl.appendChild(passengerEl);
        elevatorEl.appendChild(floorNum);
        shaft.appendChild(elevatorEl);
        elevatorsPanel.appendChild(shaft);
    }
}

// ---------- 乘客管理 ----------
function createPassengerDOM(passenger) {
    const el = document.createElement('div');
    el.className = 'passenger';
    el.style.backgroundColor = passenger.color;
    
    const tooltip = document.createElement('span');
    tooltip.className = 'passenger-tooltip';
    tooltip.textContent = `目标: ${passenger.targetFloor}F`;
    el.appendChild(tooltip);
    
    el.addEventListener('mouseenter', () => tooltip.style.display = 'block');
    el.addEventListener('mouseleave', () => tooltip.style.display = 'none');
    
    return el;
}

function generatePassenger(floor) {
    let targetFloor;
    do {
        targetFloor = Math.floor(Math.random() * FLOORS) + 1;
    } while (targetFloor === floor);
    
    const passenger = {
        id: state.nextPassengerId++,
        currentFloor: floor,
        targetFloor,
        state: 'waiting',
        color: `hsl(${Math.random() * 360}, 70%, 60%)`,
    };
    
    state.passengers.push(passenger);
    state.waitingQueues[floor].push(passenger.id);
    
    const el = createPassengerDOM(passenger);
    passenger.element = el;
    
    const waitingArea = document.getElementById(`waiting-area-${floor}`);
    waitingArea.appendChild(el);
    
    updateFloorStatus(floor);
}

function updateFloorStatus(floor) {
    const count = state.waitingQueues[floor].length;
    const el = document.getElementById(`wait-count-${floor}`);
    if (el) el.textContent = `${count}人`;
}

// ---------- 电梯调度 ----------
function findNearestRequest(elevator) {
    let bestFloor = null;
    let bestDistance = Infinity;
    
    for (let floor = 1; floor <= FLOORS; floor++) {
        if (state.waitingQueues[floor].length === 0) continue;
        
        const alreadyAssigned = state.elevators.some(e =>
            e.id !== elevator.id &&
            e.state === 'movingPickup' &&
            e.targetFloor === floor
        );
        if (alreadyAssigned) continue;
        
        const distance = Math.abs(elevator.currentFloor - floor);
        if (distance < bestDistance) {
            bestDistance = distance;
            bestFloor = floor;
        }
    }
    
    return bestFloor ? { floor: bestFloor } : null;
}

// ---------- 电梯到达处理 ----------
function handleElevatorArrival(elevator) {
    if (elevator.state === 'movingPickup') {
        const floor = Math.round(elevator.currentFloor);
        const queue = state.waitingQueues[floor];
        
        if (queue.length === 0) {
            elevator.state = 'idle';
            elevator.targetFloor = null;
            updateElevatorStyle(elevator);
            return;
        }
        
        const passengerId = queue.shift();
        const passenger = state.passengers.find(p => p.id === passengerId);
        if (!passenger) {
            elevator.state = 'idle';
            elevator.targetFloor = null;
            updateElevatorStyle(elevator);
            return;
        }
        
        passenger.state = 'inElevator';
        elevator.passenger = passenger;
        
        // 移除等待区DOM
        if (passenger.element) passenger.element.remove();
        
        // 在电梯中显示乘客
        elevator.passengerElement.style.backgroundColor = passenger.color;
        elevator.passengerElement.style.display = 'block';
        
        // 添加tooltip到电梯乘客
        const tooltip = document.createElement('span');
        tooltip.className = 'passenger-tooltip';
        tooltip.textContent = `目标: ${passenger.targetFloor}F`;
        elevator.passengerElement.appendChild(tooltip);
        elevator.passengerElement.addEventListener('mouseenter', () => tooltip.style.display = 'block');
        elevator.passengerElement.addEventListener('mouseleave', () => tooltip.style.display = 'none');
        
        elevator.state = 'movingDropoff';
        elevator.targetFloor = passenger.targetFloor;
        
        updateFloorStatus(floor);
        updateElevatorStyle(elevator);
        
    } else if (elevator.state === 'movingDropoff') {
        const passenger = elevator.passenger;
        if (passenger) {
            passenger.state = 'arrived';
        }
        
        elevator.passengerElement.style.display = 'none';
        elevator.passengerElement.style.backgroundColor = 'transparent';
        // 清除tooltip
        const tooltip = elevator.passengerElement.querySelector('.passenger-tooltip');
        if (tooltip) tooltip.remove();
        
        elevator.passenger = null;
        elevator.state = 'idle';
        elevator.targetFloor = null;
        updateElevatorStyle(elevator);
    }
    updateElevatorVisual(elevator);
}

// ---------- 电梯视觉 ----------
function updateElevatorVisual(elevator) {
    const y = getFloorY(elevator.currentFloor) - ELEVATOR_HEIGHT / 2;
    elevator.element.style.top = `${y}px`;
    const floorNum = Math.max(1, Math.min(FLOORS, Math.round(elevator.currentFloor)));
    const label = elevator.element.querySelector('.elevator-floor-num');
    if (label) label.textContent = `${floorNum}F`;
}

function updateElevatorStyle(elevator) {
    elevator.element.classList.remove('elevator-idle', 'elevator-busy');
    if (elevator.state === 'idle') {
        elevator.element.classList.add('elevator-idle');
    } else {
        elevator.element.classList.add('elevator-busy');
    }
}

// ---------- 主循环 ----------
function loop(time) {
    const deltaTime = Math.min((time - state.lastTime) / 1000, 0.1);
    state.lastTime = time;
    
    // 空闲电梯调度
    state.elevators.forEach(elevator => {
        if (elevator.state === 'idle') {
            const request = findNearestRequest(elevator);
            if (request) {
                elevator.targetFloor = request.floor;
                elevator.state = 'movingPickup';
                updateElevatorStyle(elevator);
                
                // 如果已在目标楼层，立即到达
                if (Math.abs(elevator.currentFloor - elevator.targetFloor) < 0.01) {
                    handleElevatorArrival(elevator);
                }
            }
        }
    });
    
    // 电梯移动
    state.elevators.forEach(elevator => {
        if (elevator.state === 'movingPickup' || elevator.state === 'movingDropoff') {
            const direction = Math.sign(elevator.targetFloor - elevator.currentFloor);
            elevator.currentFloor += direction * ELEVATOR_SPEED * deltaTime;
            
            if ((direction > 0 && elevator.currentFloor >= elevator.targetFloor) ||
                (direction < 0 && elevator.currentFloor <= elevator.targetFloor)) {
                elevator.currentFloor = elevator.targetFloor;
                handleElevatorArrival(elevator);
            }
            
            updateElevatorVisual(elevator);
        }
    });
    
    // 更新所有电梯视觉（确保位置正确）
    // state.elevators.forEach(updateElevatorVisual);
    
    requestAnimationFrame(loop);
}

// ---------- 初始化 ----------
function init() {
    // 初始化等待队列
    for (let f = 1; f <= FLOORS; f++) {
        state.waitingQueues[f] = [];
    }
    
    setupDOM();
    
    // 创建电梯
    for (let i = 0; i < ELEVATOR_COUNT; i++) {
        const elevator = {
            id: i,
            currentFloor: 1,
            targetFloor: null,
            state: 'idle',
            passenger: null,
            element: document.getElementById(`elevator-${i}`),
            passengerElement: document.getElementById(`elevator-${i}-passenger`),
        };
        state.elevators.push(elevator);
        updateElevatorVisual(elevator);
        updateElevatorStyle(elevator);
    }
    
    state.lastTime = performance.now();
    requestAnimationFrame(loop);
}

window.addEventListener('DOMContentLoaded', init);
```

这里有一个问题：`handleElevatorArrival`中在`movingDropoff`分支里清除了tooltip。但是当乘客一直在电梯中，而电梯移动时，tooltip显示应该仍然有效。实际上，tooltip是`elevator-passenger`元素的一部分，当`display:none`时整个隐藏。

但是，`elevator-passenger`元素本身是一个小圆点，当没有乘客时display:none。当有乘客时显示。这样可以。

不过，我在`handleElevatorArrival`的`movingPickup`分支中添加了tooltip到`elevatorPassenger`元素。当`elevatorPassenger`显示时，tooltip会在hover时显示。这没问题。

但是，每次`movingPickup`分支执行时，都会在`elevator-passenger`上添加新的tooltip，如果之前有残留，会重复添加。我在`movingDropoff`分支中清理了tooltip。所以不会重复。

好的，但还有一个小问题：`elevator-passenger`元素在`handleElevatorArrival`的`movingDropoff`分支中被`display:none`隐藏，但在下次有乘客进入时又显示。工具提示的DOM在隐藏期间仍然存在。所以我在`movingDropoff`分支中清除了tooltip。这样当新乘客进入时，会添加新的tooltip。没问题。

还有一个问题：乘客的`element`属性。在`generatePassenger`中设置了`passenger.element = el`，在`handleElevatorArrival`的`movingPickup`分支中，使用`passenger.element.remove()`移除了等待区DOM。但当乘客到达后，`passenger.element`仍然指向已删除的DOM。这可能导致问题。

让我在乘客到达后设置`passenger.element = null`。

实际上，在乘客到达后，我们不再需要`passenger.element`，所以可以设为null。

好的，让我修正。

**最终修正后的代码：**

在`handleElevatorArrival`的`movingDropoff`分支中：
```javascript
if (passenger) {
    passenger.state = 'arrived';
    passenger.element = null;
}
```

还有，在`movingPickup`分支中，如果`passenger.element`存在则移除。
```javascript
if (passenger.element) passenger.element.remove();
```

好。

**关于性能：**

主循环每帧运行，处理电梯移动。由于电梯数量少，性能不成问题。

**关于tooltip的显示：**

CSS中`passenger-tooltip`默认`display:none`，鼠标进入时设置`display:block`，离开时设置`display:none`。使用`mouseenter/mouseleave`。

**让我也处理一种情况：**

当电梯到达接客楼层时，如果楼层有多个等待乘客，其中一个乘客上电梯。其他乘客留在原地。这已经由`shift()`保证。

**好的，最终代码即将完成。**

让我写HTML部分：

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>电梯模拟器</title>
<style>
/* ...CSS... */
</style>
</head>
<body>
<div id="app">
    <h1>🏢 电梯模拟器</h1>
    <div id="building">
        <div id="floors-panel"></div>
        <div id="elevators-panel"></div>
    </div>
    <div class="info-bar">
        <span id="total-passengers">总乘客: 0</span>
        <span id="served-passengers">已送达: 0</span>
    </div>
</div>
<script>
/* ...JS... */
</script>
</body>
</html>
```

让我添加一个信息栏显示总乘客数和已送达数。这不算复杂。

让我在状态中加入计数：
```javascript
let totalGenerated = 0;
let totalServed = 0;
```

在generatePassenger中totalGenerated++，在dropoff中totalServed++。

然后在UI中更新。

好的。

**现在让我把完整的代码写出来。**

我需要调整一些CSS细节以确保视觉美观。

让我在代码中加入细致的设计：

- 楼层行hover效果
- 电梯指示灯的样式
- 乘客颜色透明度

让我在CSS中完善。

**关于工具提示的改进：**

为了效果更好，我把tooltip设计为一个浮层，显示目标楼层，并且与乘客颜色匹配。

不需要过度设计，就简单的白底黑字或黑底白字。

**好的，让我写最终代码。这将是一个很长的文件，但我会确保它完整且可运行。**

让我按照顺序写：

1. HTML结构
2. CSS样式
3. JavaScript逻辑

我会仔细测试逻辑。

让我也加入电梯的"门"的视觉元素：在电梯元素中，用一条线表示门缝。

```css
.elevator {
    background: linear-gradient(90deg, #3d4a5d 0%, #3d4a5d 48%, #2a2e38 48%, #2a2e38 52%, #3d4a5d 52%);
}
```

这会让电梯看起来有门。

这样电梯中间有一条深色缝隙，看起来像两扇门。

好的，我将在CSS中实现。

另外，电梯的当前楼层可以用一个小的楼层指示器显示在轨道旁边。但为了简洁，我就在电梯内部显示。

让我在电梯内部下方显示一个小标签，标注电梯编号和当前楼层。

```html
<div class="elevator-label">1号 · 1F</div>
```

好了，让我写完整代码。

实际上，我已经设计得差不多了。让我集中精力把代码写出来。

最后一个注意事项：当用户快速点击生成按钮时，多个乘客会同时出现在等待区。我们的代码使用`flex-wrap`，所以等待区可以容纳多个乘客。

但如果等待区乘客过多，可能会超出楼层行的高度（100px）。我们使用`flex-wrap`和`align-items: center`，但高度仍可能超出。不过对于演示来说，可以接受。

让我把等待区设置`overflow: hidden`或`max-height: 90px`，并让多余的乘客显示为小点。嗯，为了简洁，不处理过多的情况。用户不会在短时间内生成过多乘客。

**好的，最终代码开始。**

我将在代码中注意以下几点：

- 所有变量用`const`或`let`声明，避免全局污染。
- 使用事件委托或直接绑定。
- 避免频繁的DOM查询，在初始化时保存引用。

**让我把代码写出来吧。这会花费一些时间，但我要确保质量。**

我这里有个考虑：`handleElevatorArrival`中`movingPickup`分支里，如果队列为空，电梯状态变为idle。但此时电梯可能在楼层之间（如果targetFloor是浮点数），但这种情况不会发生，因为targetFloor总是整数，且到达时我们已经把currentFloor设为targetFloor。

好的，没问题。

还有一个边界情况：电梯的targetFloor在movingDropoff时设置为乘客的目标楼层。乘客目标楼层是1-6之间的整数。没问题。

让我在代码中处理`findNearestRequest`时，如果电梯已经在某个楼层且该楼层有等待乘客，但该楼层已被其他电梯预约，所以跳过。如果电梯就在该楼层，距离为0，应该优先选择。代码中`distance = 0`，会是bestDistance=0，所以会被选中。

嗯，但注意：`alreadyAssigned`检查时，如果另一部电梯也已经预约了该楼层，则跳过。但如果当前电梯就在该楼层，而另一部电梯也在来的路上，那当前电梯应该可以接客。所以`alreadyAssigned`只排除目标楼层等于该楼层的其他电梯。但目标楼层相同意味着已经在处理该楼层的乘客，所以当前电梯不应该去。

但如果另一部电梯刚从该楼层接走一个乘客（状态为movingDropoff），那它不在movingPickup状态，targetFloor是乘客的目标楼层，不是该楼层。所以不会阻塞。

好的，`alreadyAssigned`的检查是合理的。

**让我考虑一个更复杂的情况：**

楼层3有2个等待乘客。电梯A在1楼，电梯B在2楼。

- 电梯A先调度（id=0先遍历），发现楼层3有等待乘客，targetFloor=3，state='movingPickup'。
- 电梯B后调度，发现楼层3已有电梯A前往（alreadyAssigned=true），跳过。
- 电梯B没有其他请求，保持idle。

A到达3楼，接走一个乘客，去目标楼层。此时3楼还有1个等待乘客。

A离开后，B空闲，B调度时看到3楼还有等待乘客，于是B去3楼。

这没问题。

但如果电梯B在1楼，电梯A在2楼，都空闲。楼层3有2个乘客。

- 电梯A（id=0）先调度，去3楼。
- 电梯B（id=1）后调度，发现3楼已被A预约，跳过。B保持idle。

这正确。A到达后接走一个，B才会接下一个。

好的，这个调度算法是合理的。

**最终代码：**

现在，让我写完整代码。我们开始吧。

实际上，我意识到一个潜在的问题：在`loop`函数中，我们先调度空闲电梯，然后移动电梯。调度过程中，如果电梯被分配了任务，它的state变成movingPickup。然后在同一次循环中，移动电梯的代码会执行该电梯的移动。这可能没问题，因为移动是基于deltaTime的，很小的增量。

但有一个小问题：如果电梯刚被分配任务，`currentFloor`是之前的浮点数，可能不是整数。例如电梯在1.0，分配任务去3楼。direction=1，移动0.03层。没问题。

如果电梯在1.0，分配任务去1楼（如果同一楼层有乘客）。这时我们会立即调用handleElevatorArrival处理。没问题。

好的。

**让我也加入一个小的细节：** 当楼层等待区有乘客时，楼层行会有一个背景高亮，表示有乘客等待。

```css
.floor-row.has-passengers {
    background: rgba(78, 205, 196, 0.05);
}
```

在更新楼层状态时添加/删除这个class。

好的，实现之。

**最终代码编写开始。**

我需要控制代码量，同时保证功能完整。让我尽量写得简洁一些。

让我用更精简的方式处理tooltip：不使用DOM事件绑定，而是使用CSS的`:hover`和`::before/::after`伪元素？不行，因为tooltip内容需要动态设置。

所以还是用JS绑定事件。

让我内联事件处理：

```javascript
el.addEventListener('mouseenter', () => {
    tooltip.style.display = 'block';
});
el.addEventListener('mouseleave', () => {
    tooltip.style.display = 'none';
});
```

这已经够简洁了。

好，写吧。

让我把代码组织好，一次写完。

我会在HTML中嵌入所有代码。文件的最后，我将提供运行说明和测试说明。

但用户只要求"直接完成任务并交付可运行产物；同时提供运行方式和必要的测试说明"。这些说明可以写在HTML文件的注释中，或者在交付时写在旁边。由于用户要求"只给我一个文件"，说明应该包含在文件中。让我在HTML中添加注释说明。

让我在HTML底部添加一个说明部分，或使用注释。

好的，我会在HTML文件的注释中写运行方式和测试说明。

**最终HTML文件：**

让我仔细写完。

我现在就开始码代码。这会是一个较长的文件，但我会确保它完整。

以下是我将生成的完整代码的骨架。我会仔细处理每个部分。

代码中的CSS，让我设计得更现代一些。

好的，开始。
