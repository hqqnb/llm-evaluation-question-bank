用户要求制作一个电梯模拟器，具体要求如下：
1. 允许用户在不同楼层生成一定数量的人。
2. 每部电梯一次只能搭载一个人（单载客量）。
3. 总共有三部电梯。
4. 一名乘客上电梯后，其他乘客应留在原地等待；当下一部电梯到达时空闲时，他们再依次上电梯。
5. 每个人都有一个随机分配的目标楼层。
6. 鼠标悬停在人物上时，显示一个包含其目标楼层的小提示框（tooltip）。
7. 整体具有流畅的动画、良好的视觉效果，保持简洁。
8. 全部使用 HTML、CSS 和 JavaScript 构建，只给一个文件。不要查看其他文件，直接创建。
9. 输出要求：直接完成任务并交付可运行产物；同时提供运行方式和必要的测试说明。

分析需求：
- 楼层数量：为了简单，假设有5层或10层？用户没说，我可以设定一个固定的楼层数，比如10层。
- 界面布局：
  - 左侧或上方：控制面板，用于选择楼层并生成指定数量的人。
  - 右侧或下方：模拟区域，显示楼层（水平或垂直排列，垂直比较符合直觉），电梯井道，电梯轿厢，以及等待的人物。
- 电梯逻辑：
  - 3部电梯。
  - 电梯状态：空闲、运行（上行、下行）、目标楼层、当前楼层、内部乘客。
  - 乘客逻辑：生成在特定楼层，处于等待状态。当空闲电梯到达该楼层且电梯为空时，乘客进入电梯（一次一个，虽然电梯一次只能搭载一个人，但如果有多个等待者，上一个走了下一个可能要等下一趟或者同方向？不对，需求说“一名乘客上电梯后，其他乘客应留在原地等待；当下一部电梯（三部中的任意一部）到达时，他们再依次上电梯” 或者 意思是乘客依次上电梯（如果电梯容量是1，那上了1个之后，电梯就走了，剩下的人等下一部电梯？或者同一部电梯回来？但需求说“下一部电梯（三部中的任意一部）到达时”，可能意味着由于容量是1，一个电梯只能带一个人，所以一个电梯来了，带走一个人，剩下的人等另外的电梯或者这部电梯回来？等一下，“一名乘客上电梯后，其他乘客应留在原地等待；当下一部电梯（三部中的任意一部）到达时，他们再依次上电梯。” 这句话可能指的是：电梯到达，带一个人走。剩下的人等下一趟（可能是别的电梯，也可能是同一部电梯下次来）。而且“每部电梯一次只能搭载一个人”。也就是说，每部电梯容量为1。所以如果一个楼层有5个人，需要5趟电梯（不一定是5部不同的，但每次只能上1人）。
  - 调度算法：简单的就近原则或者轮询。当有多部电梯空闲时，派最近的。如果没有空闲，就排队（乘客等待，电梯完成当前任务后响应呼叫）。为了简化，每一层都有上行/下行呼叫（或者说只要有等待乘客就呼叫）。电梯扫描或者找最近的等待乘客。
  - 乘客进入电梯后，电梯获得该乘客的目标楼层，然后前往该楼层。到达后，乘客离开（走出去消失），电梯变为空闲。
- 动画：使用 CSS transitions 实现电梯的平滑移动，以及乘客的出现/消失。
- Tooltip：用 title 属性，或者用 CSS/JS 实现自定义 tooltip。为了视觉效果，可以用绝对定位的 div 跟随鼠标，或者简单的 CSS `:hover` 显示 `::after` 伪元素，包含目标楼层。
- 单文件：包含 `<style>` 和 `<script>` 内联。

具体设计：
- 楼层数 N = 6（例如）。
- 数据结构：
  - `floors`: 数组或固定数量，每个楼层有等待队列（数组）。
  - `elevators`: 数组，长度3。每个电梯有：当前楼层（浮点数用于动画，或者整数用于逻辑，CSS 用 top 百分比）、状态（idle, moving, doorOpen）、目标楼层（如果有）、乘客（如果有，包含目标楼层）。
  - `people`: 所有乘客的集合，或者只在楼层队列和电梯里。
- 控制面板：
  - 一个 `<select>` 选择楼层。
  - 一个 `<input type="number">` 输入生成人数（比如1-10）。
  - 一个 `<button>` 点击生成。
- 渲染逻辑：
  - 可以用绝对定位。每个楼层是一条水平线。
  - 电梯井道是 3 个垂直矩形。
  - 楼层标记在左侧。
  - 等待的人在楼层线的右侧（或者左侧）排队。
  - 电梯在井道里上下移动（修改 `top` 或者 `transform: translateY`）。
- 动画与定时器：
  - 使用 `requestAnimationFrame` 或者 `setInterval` 来更新电梯位置（如果用 CSS transition，可以直接改 style 的 top，CSS 会自动动画）。
  - 电梯逻辑可以用一个游戏循环（比如每 50ms 一次）来更新状态：如果电梯空闲，检查所有楼层的等待队列，如果有人，选择最近的有人的楼层（或者分配最近的电梯），把目标设为该楼层，状态改为 moving。如果 moving 且到达目标，状态改为 doorOpen（停留一段时间，比如 1 秒），然后让人上/下，再变 idle。
  - 不过为了实现“依次上电梯”，当电梯门开时，如果本层有等待者且电梯为空，把队列第一个人移入电梯，设定电梯目标为该人的目标楼层。电梯关门，moving。等待者排队等待。

界面布局细节：
- 背景深色或浅色，简洁现代。
- 楼层：0 到 5 层（共 6 层）。0层在底部，5层在顶部。
- 电梯：宽度固定，比如 3 部电梯并排，每个宽 40px。
- 乘客：小圆点或带颜色的小方块，带有 hover tooltip。
- 控制面板在顶部或底部。

逻辑细化：
- 初始化：
  - 创建 6 层楼，每层有个 div 作为等候区。
  - 创建 3 部电梯的 div，放在对应的井道里。
- 电梯移动：
  - `elevator.currentFloor` (float), `elevator.targetFloor` (int or null), `elevator.passenger` (object or null), `elevator.state` ('idle', 'moving', 'waiting')。
  - CSS: `.elevator { transition: top 1s linear; }` （假设每层移动需要 1 秒）。
  - 实际上，如果用 CSS transition，当设置 `top` 后，需要监听 `transitionend` 或者在 JS 里用 `setTimeout` 来模拟到达时间。为了保持状态同步，最好在 JS 中用 `requestAnimationFrame` 更新 `currentFloor` 并直接设置 `style.top` (无 transition)，或者利用 `transition` 加 `setTimeout(duration)`。
  - 为简单且保证逻辑准确，我用 JS 的 `setInterval` 来跑逻辑，用 CSS transition 来跑动画。
  - 比如：电梯从 0 到 3，距离 3 层。`transition: top 3s linear`，同时 `setTimeout(3000, arrive)`。这样就不需要复杂的物理更新了。

- 乘客生成：
  - 在选定楼层 `f` 生成 `n` 个人。
  - 每个人随机目标楼层 `t` (0 到 5 且 `t != f`)。
  - 每个人是一个 div，添加到该楼层的等候区。
  - 为每个人设置 `title` 或者自定义 tooltip 显示 `目标: t 层`。

- 电梯调度：
  - 每秒钟（或者由状态改变触发），检查每部空闲电梯。
  - 如果电梯空闲，寻找需要服务的呼叫。呼叫来自于有等待乘客的楼层。
  - 计算所有“有等待乘客的楼层”中，距离该电梯当前楼层最近的楼层。
  - 如果找到了，电梯目标设为该楼层，开始移动（设置 top 和 timeout）。
  - 当电梯到达 `targetFloor`，进入 `waiting` 状态（比如停留 500ms 开门）。
  - 如果本层有等待乘客且电梯为空：
    - 取出队列第一个乘客。
    - 将该乘客的 DOM 元素从楼层移除，放到电梯里（可以用动画或者简单移动）。
    - 电梯获得乘客，记录乘客的目标楼层作为电梯的下一个 `targetFloor`。
    - 电梯状态变为 `moving`（准备带他去目标楼层）。
  - 如果电梯里有乘客且到达了乘客的目标楼层：
    - 让乘客离开（移除 DOM）。
    - 电梯变为空闲。检查本层是否还有等待乘客？由于容量是1，刚才到达时如果有等待者，因为电梯已经满了（载着上一个乘客），所以本层剩下的等待者只能等电梯下次来或者别的电梯来。这符合需求“一名乘客上电梯后，其他乘客应留在原地等待；当下一部电梯（三部中的任意一部）到达时，他们再依次上电梯。”
  - 等一下，如果电梯到达某层，而电梯是空的，且该层有人等待，则接走一人，并立刻（或稍作停顿后）前往该乘客的目标楼层。这符合逻辑。

多电梯协调：
- 如果多部电梯同时空闲，可能会奔向同一层。这没关系，但当第一部到达时接走一人，第二部到达时接走下一个，第三部到达接走第三个。如果乘客数少于空闲电梯数，多余的电梯到达后发现没人了（被前面的电梯接走了，或者本来就没人），就变为空闲重新寻找呼叫。
- 为了避免多部电梯都去同一层导致“空跑”，可以实现一个简单的分配：每个有等待乘客的楼层，只分配给一部最近的电梯（标记该层已被某电梯响应）。等一下，需求没要求特别智能的调度，只要能用就行。但是如果有 3 部电梯都去 5 楼，只有一个人在 5 楼等，那两部电梯白跑。稍微优化一下：每个楼层记录 `assignedElevator`（如果有人等待且有电梯被分配）。当电梯空闲找目标时，只找没有被分配出去的等待楼层（或者如果都被分配了，且自己距离最近，可以抢？不用那么复杂，简单点：遍历楼层，如果有等待乘客且楼层没有被分配给任何电梯，就分配给当前这部电梯。如果有等待乘客但已经分配了，就不分配。但如果没有任何未分配的楼层，再找最近的？）。其实最简单的做法是：电梯找最近的等待楼层，直接去。到了之后如果没人（被其他电梯接走了），就重新找。这样会有少量空跑，但逻辑简单且符合物理世界的“傻瓜”调度。因为是模拟器，效果上能看到电梯移动就行。为了表现更好，可以加个“呼叫”状态：当乘客在某层生成，该层发出呼叫；电梯A认领该呼叫（即使还没到），其他电梯就不认领了；电梯A到达并接走人后，该呼叫取消；如果电梯A在途中乘客被别人接走了（不会发生，因为只有认领的才去），或者电梯A到达发现人没了（说明逻辑有问题）。
- 为了严格保证“依次上电梯”且避免空跑：当乘客在楼层 `f` 时，楼层 `f` 处于 `waiting` 状态。我们给每个电梯一个 `targetFloor`。每次电梯空闲时，寻找所有有等待乘客的楼层，如果没有电梯的 `targetFloor` 指向该楼层，则该空闲电梯可以认领最近的楼层。这样每有一个等待乘客（或每一层有等待乘客），最多只有一部电梯去接。但如果该层有 5 个乘客，一部电梯接走 1 个，剩下 4 个还在等待。此时该层又变成“无人认领的等待楼层”，下一部空闲电梯会来接。这完美符合需求，且几乎无空跑！
- 总结调度：
  - 楼层有等待队列，如果队列长度 > 0，则该楼层需要服务。
  - 遍历所有电梯，如果电梯 `state === 'idle'`：
    - 找出所有“需要服务且未被其他电梯认领（即没有其他电梯的 targetFloor 等于该楼层且该电梯正在移动/已认领）”的楼层。
    - 如果有，选最近的一个楼层作为 `targetFloor`，状态设为 `moving`。
    - 如果没有未被认领的楼层，但有需要服务的楼层（意味着所有楼层都被认领了，但当前电梯闲着），那它就不动作，等会儿再看（或者去最近的认领楼层凑热闹？不，就原地等，因为被认领的楼层迟早会被服务，服务完如果还有人就会释放认领状态）。
  - 等等，如果一部电梯认领了楼层 5，到达后接走 1 人，此时楼层 5 还有 4 人，电梯刚离开去送人，楼层 5 仍然是需要服务的，此时它没有被任何电梯认领（因为接人的电梯已经走了）。下一部空闲电梯就会来认领楼层 5。这是完美的！

数据结构与状态：
```javascript
const NUM_FLOORS = 6; // 0 to 5
const NUM_ELEVATORS = 3;
let elevators = [];
let floors = []; // floors[f] = array of passengers

// Passenger: { id, targetFloor, element }
// Elevator: { id, currentFloor, targetFloor, state, passenger, element, doorTimer }
```

具体交互和DOM：
- `building` div 包含 `floors` 和 `elevators`。
- `floors` 区域：每层一个 `floor-row`，里面有楼层号和乘客等待区。
- `elevators` 区域：绝对定位的 3 个 `elevator` 方块。
- 楼层坐标：假设楼层高度 `FLOOR_HEIGHT = 60px`。总高度 360px。
- 最顶部是 5 楼，最底部是 0 楼（或者反过来，物理上 0 楼在底部）。
- 使用 CSS 的 `bottom` 属性定位，或者 `top` 都行。`top = (NUM_FLOORS - 1 - currentFloor) * FLOOR_HEIGHT`。

乘客生成 UI：
- 顶部控制栏：
  - `<label>楼层: <select id="floor-select">...</select></label>`
  - `<label>人数: <input type="number" id="num-people" value="1" min="1" max="10"></label>`
  - `<button id="generate-btn">生成乘客</button>`

Tooltip：
- 乘客 div 加上 `data-tooltip="目标: 3层"`，CSS 里用 `.passenger:hover::after { content: attr(data-tooltip); position: absolute; ... }`。简单且流畅。

动画细节：
- 电梯移动：`elevator.element.style.top = ...`。CSS：`.elevator { transition: top 1s linear; }`。
- 为了让移动时间和距离成正比，可以动态设置 `transition-duration`？CSS transition 只能设一个时间，如果设 `1s linear`，那么从 0 到 5 也是 1s，太快了。
- 所以可以动态设置：`elevator.element.style.transition = 'top ' + distance + 's linear';` 然后修改 `top`。这样就实现了每层 1 秒的速度。
- 不过要注意，当 `top` 改变后，我们要在 `distance * 1000` 毫秒后触发“到达”事件。可以用 `setTimeout`。
- 电梯到达后，稍微停顿（比如 0.5s），让乘客进/出，然后关门、继续。

逻辑循环：
- 使用 `setInterval` 每 `100ms` 检查一次调度，或者使用事件驱动（状态改变时检查）。
- 其实 `setInterval` 足够简单：
```javascript
function tick() {
    elevators.forEach(elevator => {
        if (elevator.state === 'idle') {
            // 找目标
            let target = findTargetFloor(elevator);
            if (target !== null) {
                assignTarget(elevator, target);
            }
        }
    });
}
setInterval(tick, 200);
```
- `findTargetFloor`：寻找所有 `floors[f].length > 0` 的 `f`。并且这些 `f` 没有被其他电梯“认领”（即没有其他电梯的 `targetFloor === f` 且状态是 moving 或 doorOpen 且还没接走人？为了简单，只要有其他电梯的 `targetFloor === f` 就不选这个 `f`）。然后选距离 `elevator.currentFloor` 绝对值最小的 `f`。如果所有有人的楼层都被认领了，返回 `null`。
  - 等等，如果电梯A认领了楼层 5，但在去 5 楼的路上（moving），此时楼层 5 有 3 个人。电梯A到达后接走 1 人，剩下 2 人。电梯A离开去 6 楼。此时楼层 5 的人还在等，且不再被任何电梯认领，下一 tick 时，其他空闲电梯就会来认领楼层 5。这样很好。
  - 但如果电梯A正载着人经过楼层 3，而楼层 3 有人等且没被认领？电梯A正在 moving 且载着人（`passenger != null`），它不会停下（因为一次只能载一人）。它不会认领楼层 3。其他空闲电梯会来认领 3 楼。这也是正确的。

电梯状态机：
- `idle`：没有乘客，没有目标。
- `moving`：有目标楼层（可能是去接人，也可能是送人）。正在移动中。
- `doorOpen`：到达目标楼层，门开。等待 0.5s（让动画展示），处理乘客（上或下），然后设状态为 `idle` 或 `moving`（如果刚接了人，设目标为新乘客的目标楼层并准备关门移动）。

处理到达事件（到达 targetFloor）：
- 到达时（`setTimeout` 触发）：
  - `elevator.currentFloor = elevator.targetFloor;`
  - `elevator.state = 'doorOpen';`
  - 门开动画（可选，比如变色）。
  - 如果 `elevator.passenger` 存在（说明是来送人的）：
    - 乘客到达目标，移除乘客 DOM（或者做下电梯动画）。
    - `elevator.passenger = null;`
    - `elevator.targetFloor = null;`
    - 停留 0.5s 后，变为 `idle`。
  - 如果 `elevator.passenger` 为 `null`（说明是来接人的）：
    - 检查本层 `floors[elevator.currentFloor]` 是否有人。
    - 如果有人：
      - 取出队列第一个乘客（shift）。
      - 把乘客 DOM 移入电梯（视觉上，可以先从等候区移除，加进电梯 div 里，或者简单点直接隐藏/移除并在电梯里显示一个小人）。
      - `elevator.passenger = passenger;`
      - `elevator.targetFloor = passenger.targetFloor;` // 接下来去送人
      - 停留 0.5s 后，变为 `moving`，开始移动去 `passenger.targetFloor`。
    - 如果没人（被别人接走了或者本来就没有了）：
      - `elevator.targetFloor = null;`
      - 停留 0.5s 后，变为 `idle`。

乘客 DOM 处理：
- 楼层等待区：`.waiting-area` 是一个 flex 容器。等待的人作为子元素显示。
- 电梯内：`.elevator` 内部可以有一个 `.passenger-inside` 容器。当乘客进入电梯，把乘客 DOM 移入这里；离开时移除。
- 其实为了简单，乘客只用一个 div 表示。生成时 append 到 `waiting-area`。进入电梯时 `elevator.element.appendChild(passenger.element)`。到达时 `passenger.element.remove()`。由于乘客很小，CSS 可以适应父容器（电梯内部居中显示）。

调度逻辑细化：
- `findTargetFloor(elevator)`:
  - 遍历 `0` 到 `NUM_FLOORS-1`：
    - 如果有乘客：`floors[f].length > 0`
    - 检查是否被认领：是否有一个电梯 `e` (不是当前 `elevator`)，使得 `e.targetFloor === f` 且 `e.state !== 'idle'`（或者只要 `e.targetFloor === f` 就行，因为 `idle` 不会设 `targetFloor`）。
    - 如果没有被认领，记录这个距离 `Math.abs(f - elevator.currentFloor)`。
  - 选距离最小的 `f`。返回 `f` 或 `null`。
  - 注意：如果当前电梯正在移动去接人，它的 `targetFloor` 就是那个楼层，所以不会被自己或其他人重复认领（因为 `targetFloor === f` 被排除了）。

移动实现：
```javascript
function assignTarget(elevator, floor) {
    elevator.targetFloor = floor;
    elevator.state = 'moving';
    let distance = Math.abs(floor - elevator.currentFloor);
    let duration = distance * 1.0; // 1 sec per floor
    elevator.element.style.transition = `top ${duration}s linear`;
    elevator.element.style.top = getTopForFloor(floor);
    
    setTimeout(() => {
        arriveAtFloor(elevator);
    }, duration * 1000);
}
```

到达处理：
```javascript
function arriveAtFloor(elevator) {
    elevator.currentFloor = elevator.targetFloor;
    elevator.state = 'doorOpen';
    elevator.element.classList.add('door-open'); // 视觉提示
    
    setTimeout(() => {
        elevator.element.classList.remove('door-open');
        if (elevator.passenger) {
            // 送人到达
            elevator.passenger.element.remove();
            elevator.passenger = null;
            elevator.targetFloor = null;
            elevator.state = 'idle';
        } else {
            // 接人
            let queue = floors[elevator.currentFloor];
            if (queue.length > 0) {
                let p = queue.shift();
                elevator.element.appendChild(p.element); // 从楼层 DOM 移到电梯 DOM
                elevator.passenger = p;
                elevator.targetFloor = p.targetFloor;
                // 稍微停留后去目标
                setTimeout(() => {
                    // 去乘客目标
                    let dest = elevator.targetFloor;
                    let distance = Math.abs(dest - elevator.currentFloor);
                    let duration = distance * 1.0;
                    elevator.element.style.transition = `top ${duration}s linear`;
                    elevator.element.style.top = getTopForFloor(dest);
                    setTimeout(() => {
                        arriveAtFloor(elevator); // 递归调用，此时 passenger 有值，会走送人逻辑
                    }, duration * 1000);
                }, 500); // 开门停留 0.5s
            } else {
                elevator.targetFloor = null;
                elevator.state = 'idle';
            }
        }
    }, 500); // 到达后停留 0.5s
}
```

等等，上面的 `assignTarget` 和 `arriveAtFloor` 中“接人后去目标”的逻辑，把移动写死在 `setTimeout` 里了。其实可以把移动统一抽成一个函数 `moveTo(elevator, floor, callback)`。
```javascript
function moveTo(elevator, floor, callback) {
    elevator.targetFloor = floor;
    elevator.state = 'moving';
    let distance = Math.abs(floor - elevator.currentFloor);
    let duration = distance * 1.0; // 1s per floor
    elevator.element.style.transition = `top ${duration}s linear`;
    elevator.element.style.top = getTopForFloor(floor);
    setTimeout(() => {
        elevator.currentFloor = floor;
        callback();
    }, duration * 1000);
}
```
然后在 `tick` 逻辑或者 `idle` 检查中调用 `moveTo` 去接人：
```javascript
if (elevator.state === 'idle') {
    let target = findTargetFloor(elevator);
    if (target !== null) {
        moveTo(elevator, target, () => {
            // 到达接人楼层
            elevator.state = 'doorOpen';
            setTimeout(() => {
                let queue = floors[elevator.currentFloor];
                if (queue.length > 0) {
                    let p = queue.shift();
                    elevator.element.appendChild(p.element);
                    elevator.passenger = p;
                    // 接着去送人
                    moveTo(elevator, p.targetFloor, () => {
                        // 到达送人楼层
                        elevator.state = 'doorOpen';
                        setTimeout(() => {
                            p.element.remove();
                            elevator.passenger = null;
                            elevator.state = 'idle';
                        }, 500);
                    });
                } else {
                    // 没人，变 idle
                    elevator.state = 'idle';
                }
            }, 500);
        });
    }
}
```
这样逻辑清晰很多。但是注意：如果一个电梯在 `idle` 时找不到目标，它就在 `tick` 里啥也不做。下一个 `tick` (200ms后) 再找。这没问题。
还有个小问题：在 `moveTo` 的回调里，我们设置了 `state = 'doorOpen'`，然后 500ms 后处理。但是在这 500ms 内，`tick` 可能会运行，如果 `elevator.state` 不是 `idle`，`tick` 不会给它分配新目标，这很好。但是 `doorOpen` 状态需要明确，避免被看作是 `idle`。所以 `state` 检查只用 `idle` 就够了。

但是有一个问题：当电梯正在 `moving` 去接人时，其他电梯的 `tick` 会发现那个楼层被认领了吗？
在我们的 `findTargetFloor` 里，我们排除了“有其他电梯的 `targetFloor === f` 且该电梯不是 idle”的楼层。而在 `moveTo` 里，我们把 `targetFloor` 设为了目标楼层，所以此时该楼层已经被认领了。这很好，避免了多个电梯去同一层接仅有的一个人。
如果电梯A到达接人楼层，接走了一个人，然后 `moveTo` 去送人，此时它的 `targetFloor` 改成了乘客的目标楼层。那么原先接人的楼层就不再被任何电梯认领了（如果还有等待乘客的话）。这完全正确！下一 tick 其他空闲电梯就能认领那个楼层了。

楼层坐标计算：
HTML 结构：
```html
<div class="building">
    <div class="floors">
        <!-- 对于每个楼层，从顶到底是 5,4,3,2,1,0 -->
        <div class="floor" id="floor-5">
            <div class="floor-label">5</div>
            <div class="waiting-area"></div>
        </div>
        ...
        <div class="floor" id="floor-0">...</div>
    </div>
    <div class="elevators">
        <div class="elevator-shaft"></div>
        <div class="elevator-shaft"></div>
        <div class="elevator-shaft"></div>
        <div class="elevator" id="elev-0"></div>
        <div class="elevator" id="elev-1"></div>
        <div class="elevator" id="elev-2"></div>
    </div>
</div>
```
CSS 定位：
`building` 是 `position: relative`，宽高固定。
`floors` 是正常文档流（或者绝对定位），每层高度比如 `60px`。
`elevators` 绝对定位在右侧，`top: 0`, `right: 0`, `bottom: 0`, `width: 200px`。里面包含 3 个井道（竖线或背景）和 3 个电梯方块。
电梯方块的 `top` 也是按 `(NUM_FLOORS - 1 - currentFloor) * 60px` 计算（如果 0 楼在最下面，那 0 楼 top 是 `(5-0)*60 = 300px`，5楼 top 是 0px）。这要求 `building` 或 `elevators` 容器高度是 `6 * 60 = 360px`。

让我精确设计坐标：
- `FLOOR_HEIGHT = 60` px。
- 楼层总数 `N = 6` (0~5)。
- 总高度 `360px`。
- `floor` 的顺序：DOM 里从上到下写 5, 4, 3, 2, 1, 0，这样符合自然视觉，不需要反向 flex。每 `floor` 高度 `60px`。
- 电梯容器 `elevators` 和 `floors` 并排（比如用 flex），高度都是 `360px`。
- 电梯 `elevator` 元素 `position: absolute`，`width: 50px`，`height: 50px`，`margin: 5px`。
- 电梯所在的井道：假设 3 个电梯并排，每个占 60px 宽。
- `getTopForFloor(floor)` 函数：`return (N - 1 - floor) * FLOOR_HEIGHT + 'px';`
  例如 floor=5 返回 0px，floor=0 返回 300px。正确。
- 乘客在等待区 `.waiting-area` 用 `display: flex; gap: 4px; align-items: center;` 排队。

乘客样式：
- `passenger` 是一个小圆圈，宽高 `24px`，背景色随机或固定，圆角 `50%`，`position: relative`（为了 tooltip 绝对定位）。
- Tooltip：
```css
.passenger::after {
    content: attr(data-tooltip);
    position: absolute;
    bottom: 120%;
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
}
.passenger:hover::after {
    opacity: 1;
}
```

乘客目标：
- 生成乘客时：`let target = Math.floor(Math.random() * N); while(target === currentFloor) target = Math.floor(Math.random() * N);`
- `data-tooltip = "目标: " + target + "层"`

视觉美观：
- 现代简洁风格：背景 `#f0f2f5`，建筑区域白色卡片，阴影，圆角。
- 电梯井道：浅灰色背景，边框。
- 电梯：蓝色背景，有光泽（比如 `box-shadow` 或者渐变），门打开效果可以用伪元素或者改变背景色（比如到了变绿，或者门打开变浅）。
- 等待区：在楼层左侧，电梯在右侧，所以 `floor` 用 `display: flex`，左边 `floor-label` 和 `waiting-area`，右边留给电梯？不，最好电梯区域和楼层区域是分开的两列。左边楼层（包含标签和等待区），右边电梯井。
- 具体布局：
```html
<div class="simulator">
    <div class="controls">...</div>
    <div class="building">
        <div class="floors">
            <!-- 楼层 -->
        </div>
        <div class="elevators">
            <!-- 井道背景和电梯 -->
        </div>
    </div>
</div>
```
CSS:
```css
.building { display: flex; height: 360px; gap: 20px; }
.floors { flex: 1; display: flex; flex-direction: column; }
.floor { height: 60px; display: flex; align-items: center; border-bottom: 1px solid #ddd; }
.elevators { position: relative; width: 200px; background: #eaeaea; border: 1px solid #ccc; }
.elevator-shaft { position: absolute; top: 0; bottom: 0; width: 60px; border-right: 1px solid #ccc; }
/* 三个井道位置：0, 60, 120 */
.elevator { position: absolute; width: 50px; height: 50px; left: 5px; background: #3498db; border-radius: 6px; transition: top 1s linear, background 0.3s; }
```
这样电梯 0 在 `left: 5px`，电梯 1 在 `left: 65px`，电梯 2 在 `left: 125px`。

等待区 `.waiting-area`：
```css
.waiting-area { display: flex; gap: 6px; margin-left: 10px; }
.passenger { width: 24px; height: 24px; background: #e74c3c; border-radius: 50%; cursor: pointer; }
```
乘客进入电梯后，他的 DOM 被移动到了 `.elevator` 里。由于 `.elevator` 也是 `display: flex; justify-content: center; align-items: center;`，乘客会在电梯中间显示。不过电梯可能还要显示一点别的？不用，就显示乘客在电梯里即可。

关于“一名乘客上电梯后，其他乘客应留在原地等待；当下一部电梯（三部中的任意一部）到达时，他们再依次上电梯。”
- 这已经被自然实现了：因为一部电梯一次只接一个人（容量=1），接走后电梯状态变为 moving（去送人），此时 `targetFloor` 改为乘客目的地，所以它离开了。本层剩下的乘客继续留在 `floors[f]` 队列里。下一 tick 时，另一个空闲电梯会发现该楼层未被认领且有人，于是来接。如果只有一部电梯空闲，它送完人回来变 idle，会再次认领该楼层（如果还没被别人认领），再来接下一个。完美。

测试一下边界条件：
1. 生成了 5 个人在同一楼层（比如 3 楼）。
2. 三台电梯都在 0 楼 idle。
3. Tick 1：电梯0、1、2 都在找目标。3 楼有 5 人，未被认领。它们距离都是 3。那么三部电梯都会选 3 楼吗？
   - 假设 Tick 同时运行：电梯0先找，发现3楼无人认领，选3楼，设 `targetFloor=3`。
   - 电梯1找，发现3楼被电梯0认领了（因为 `elevator0.targetFloor === 3`），所以不选3楼。找其他楼层，没有，返回 null。
   - 电梯2同理，不选3楼。
   - 结果：只有电梯0去接人，电梯1和2原地待命。这是合理的，避免三部电梯都去接一个人。
4. 电梯0到达3楼，接走1人（还剩4人），去送人（`targetFloor` 变为乘客目的地，比如 5 楼）。此时 3 楼不再被认领（因为电梯0的 target 不是 3了）。
5. 下一个 Tick：电梯1（idle）发现 3 楼有4人且未被认领，于是去接。电梯2 也发现，但被电梯1认领了，所以不动。
6. 电梯1到达接走1人，还剩3人，去送人。
7. 然后电梯2去接…… 接着电梯0送完回来，发现3楼还有人，再去接。这就实现了“依次上电梯”，且三部电梯都会被有效利用，非常棒！

等一下，有没有可能乘客太多，且只有一部电梯，导致它来回跑？会的，但那也是符合现实的。如果3部电梯都在忙（比如分别在不同楼层送人），本层剩下的人就只能等它们送完回来。这符合逻辑。

完善 `findTargetFloor` 函数：
```javascript
function findTargetFloor(elevator) {
    let bestFloor = null;
    let minDistance = Infinity;
    
    for (let f = 0; f < NUM_FLOORS; f++) {
        if (floors[f].length > 0) {
            // 检查是否被认领
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
```
等等，如果一个楼层有人，且被认领了（`claimed === true`），那 `distance` 就不考虑了，这没问题。但是如果有人的楼层全被认领了，`bestFloor` 保持 `null`，电梯不动。这正确。

但是，有一个小漏洞：如果电梯 A 认领了楼层 F，但在去 F 的路上，有人（程序）在 F 层又生成了新乘客？F 层本来就有乘客（否则不会被认领），新乘客加入 `floors[F]` 队列。电梯 A 到达后接走一个，剩下的人（包括新乘客）还在。电梯 A 的 target 变成送人目的地，F 层释放认领。下一 tick 会有别的电梯来。这完全没问题。

乘客 DOM 移动动画：
当乘客 `shift` 并被 `appendChild` 到电梯里时，因为乘客本身没有 CSS transition（或者说我们不特别处理），它会瞬间出现在电梯里。由于电梯门“打开”（我们可以加个背景变色的类），视觉上就像上电梯了。为了流畅，可以给乘客一个 transition 吗？如果用 FLIP 技术会有点复杂。简单起见，直接 `appendChild` 然后电梯门有个开合动画就够了。
其实我们可以让乘客淡入淡出，或者直接出现。需求说“流畅的动画”，主要指电梯移动流畅，以及整体视觉效果。电梯移动我们已经用了 `transition: top 1s linear`。
门开动画：可以在 `doorOpen` 状态时给电梯加个 `.door-open` 类，比如改变背景色或者显示一个“门”分开的效果。
简单点：
```css
.elevator {
    transition: top 1s linear, background-color 0.3s;
}
.elevator.door-open {
    background-color: #2ecc71; /* 变绿表示开门/可上 */
}
```

乘客进入电梯后，因为有 `display: flex; align-items: center; justify-content: center;`，乘客会居中。

让我们梳理完整的 HTML/CSS/JS 结构。

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<title>电梯模拟器</title>
<style>
    /* 全局样式 */
    body { font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; background: #f0f2f5; margin: 0; padding: 20px; display: flex; flex-direction: column; align-items: center; }
    h1 { color: #333; }
    .controls { background: #fff; padding: 15px 20px; border-radius: 8px; box-shadow: 0 2px 8px rgba(0,0,0,0.1); margin-bottom: 20px; display: flex; gap: 15px; align-items: center; }
    .controls select, .controls input, .controls button { padding: 8px 12px; font-size: 14px; border: 1px solid #ccc; border-radius: 4px; }
    .controls button { background: #3498db; color: #fff; border: none; cursor: pointer; transition: background 0.2s; }
    .controls button:hover { background: #2980b9; }
    
    /* 建筑模拟区 */
    .building { display: flex; height: 360px; gap: 20px; background: #fff; padding: 20px; border-radius: 8px; box-shadow: 0 2px 12px rgba(0,0,0,0.1); }
    .floors { flex: 1; display: flex; flex-direction: column; border-right: 2px solid #eee; }
    .floor { height: 60px; display: flex; align-items: center; border-bottom: 1px dashed #eee; }
    .floor:last-child { border-bottom: none; }
    .floor-label { width: 40px; font-weight: bold; color: #555; text-align: center; }
    .waiting-area { display: flex; gap: 6px; margin-left: 10px; min-height: 24px; }
    
    .elevators { position: relative; width: 210px; background: #fafafa; border: 1px solid #ddd; border-radius: 4px; }
    .elevator-shaft { position: absolute; top: 0; bottom: 0; width: 70px; border-right: 1px solid #eee; }
    .elevator-shaft:last-child { border-right: none; }
    
    .elevator { position: absolute; width: 60px; height: 50px; left: 5px; background: #3498db; border-radius: 6px; transition: top 1s linear, background-color 0.3s; display: flex; justify-content: center; align-items: center; box-shadow: inset 0 0 5px rgba(0,0,0,0.2); }
    .elevator.door-open { background: #2ecc71; }
    
    /* 乘客 */
    .passenger { width: 24px; height: 24px; background: #e74c3c; border-radius: 50%; position: relative; cursor: pointer; box-shadow: 0 2px 4px rgba(0,0,0,0.2); }
    .passenger::after { content: attr(data-tooltip); position: absolute; bottom: 130%; left: 50%; transform: translateX(-50%); background: rgba(0,0,0,0.8); color: #fff; padding: 4px 8px; border-radius: 4px; font-size: 12px; white-space: nowrap; opacity: 0; pointer-events: none; transition: opacity 0.2s; z-index: 10; }
    .passenger:hover::after { opacity: 1; }
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
        <!-- 楼层动态生成（从 5 到 0） -->
    </div>
    <div class="elevators" id="elevators-container">
        <!-- 井道与电梯动态生成 -->
    </div>
</div>

<script>
    const NUM_FLOORS = 6;
    const NUM_ELEVATORS = 3;
    const FLOOR_HEIGHT = 60;

    let floors = Array.from({ length: NUM_FLOORS }, () => []);
    let elevators = [];
    let passengerIdCounter = 0;

    // 初始化 DOM
    const floorsContainer = document.getElementById('floors-container');
    const elevatorsContainer = document.getElementById('elevators-container');

    // 生成楼层 (从上到下 5 到 0)
    for (let f = NUM_FLOORS - 1; f >= 0; f--) {
        const floorDiv = document.createElement('div');
        floorDiv.className = 'floor';
        floorDiv.innerHTML = `<div class="floor-label">${f}</div><div class="waiting-area" id="waiting-${f}"></div>`;
        floorsContainer.appendChild(floorDiv);
    }

    // 生成井道和电梯
    const shafts = [];
    for (let i = 0; i < NUM_ELEVATORS; i++) {
        const shaft = document.createElement('div');
        shaft.className = 'elevator-shaft';
        shaft.style.left = (i * 70) + 'px';
        elevatorsContainer.appendChild(shaft);
    }
    for (let i = 0; i < NUM_ELEVATORS; i++) {
        const elevDiv = document.createElement('div');
        elevDiv.className = 'elevator';
        elevDiv.style.left = (i * 70 + 5) + 'px';
        elevDiv.style.top = getTopForFloor(0) + 'px'; // 初始在 0 层
        elevatorsContainer.appendChild(elevDiv);

        elevators.push({
            id: i,
            element: elevDiv,
            currentFloor: 0,
            targetFloor: null,
            state: 'idle', // idle, moving, doorOpen
            passenger: null
        });
    }

    function getTopForFloor(floor) {
        return (NUM_FLOORS - 1 - floor) * FLOOR_HEIGHT;
    }

    // 生成乘客
    document.getElementById('generate-btn').addEventListener('click', () => {
        const floor = parseInt(document.getElementById('floor-select').value);
        const num = parseInt(document.getElementById('num-people').value) || 1;
        const waitingArea = document.getElementById(`waiting-${floor}`);

        for (let i = 0; i < num; i++) {
            let target = Math.floor(Math.random() * NUM_FLOORS);
            while (target === floor) {
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
            floors[floor].push(p);
        }
    });

    // 移动电梯
    function moveTo(elevator, floor, callback) {
        elevator.targetFloor = floor;
        elevator.state = 'moving';
        let distance = Math.abs(floor - elevator.currentFloor);
        let duration = distance * 1.0; // 每层1秒
        
        elevator.element.style.transition = `top ${duration}s linear, background-color 0.3s`;
        elevator.element.style.top = getTopForFloor(floor) + 'px';

        setTimeout(() => {
            elevator.currentFloor = floor;
            callback();
        }, duration * 1000);
    }

    // 寻找目标楼层
    function findTargetFloor(elevator) {
        let bestFloor = null;
        let minDistance = Infinity;

        for (let f = 0; f < NUM_FLOORS; f++) {
            if (floors[f].length > 0) {
                // 检查是否被其他电梯认领
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

    // 主调度循环
    function tick() {
        elevators.forEach(elevator => {
            if (elevator.state === 'idle') {
                let target = findTargetFloor(elevator);
                if (target !== null) {
                    // 去接人
                    moveTo(elevator, target, () => {
                        elevator.state = 'doorOpen';
                        elevator.element.classList.add('door-open');
                        
                        setTimeout(() => {
                            elevator.element.classList.remove('door-open');
                            let queue = floors[elevator.currentFloor];
                            if (queue.length > 0) {
                                let p = queue.shift();
                                elevator.element.appendChild(p.element); // 乘客进入电梯
                                elevator.passenger = p;
                                
                                // 去送人
                                moveTo(elevator, p.targetFloor, () => {
                                    elevator.state = 'doorOpen';
                                    elevator.element.classList.add('door-open');
                                    
                                    setTimeout(() => {
                                        elevator.element.classList.remove('door-open');
                                        p.element.remove(); // 乘客离开
                                        elevator.passenger = null;
                                        elevator.state = 'idle';
                                    }, 500);
                                });
                            } else {
                                elevator.state = 'idle';
                            }
                        }, 600); // 开门停留时间
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

让我再检查一下乘客进入电梯的视觉表现。
乘客原本在 `.waiting-area` 里，是 24x24 的圆。当被 `appendChild` 到 `.elevator` 时，由于 `.elevator` 是 `display: flex; justify-content: center; align-items: center;`，乘客会显示在电梯的中央。这很好。
不过有个小问题：电梯尺寸是 60x50，乘客 24x24，能放下。如果有多个乘客在电梯里？由于电梯容量是 1，同一时间电梯最多只有 1 个乘客，所以没问题。
当乘客下电梯时，`p.element.remove()`，电梯变空。

另外，关于“一名乘客上电梯后，其他乘客应留在原地等待；当下一部电梯（三部中的任意一部）到达时，他们再依次上电梯。”
我们逻辑里，电梯0到达，接走一个，然后马上 `moveTo` 去送人。此时电梯0不再是 idle，也不会再看本层队列。本层剩下的乘客留在原地（DOM里还在 waiting-area）。然后下一 tick，电梯1发现该层未被认领且有人，于是去接，接走下一个。这就完美满足了需求。

等一下，`findTargetFloor` 里，距离计算是用 `Math.abs(f - elevator.currentFloor)`。如果有两层都有人，比如 1 楼和 5 楼，电梯在 3 楼，1 楼距离 2，5 楼距离 2，它会选哪个？因为 `distance < minDistance` 是严格小于，所以选先遍历到的（1 楼）。这无所谓，或者我们可以用 `<=`，选楼层号大的或小的。由于楼层是从 0 遍历到 5，如果是 `<=`，会选 5 楼。都行，不影响功能。

需要注意的是，一个空闲电梯在 `findTargetFloor` 时，如果有人在某层，且这层被其他电梯认领了（即其他电梯 `targetFloor === f` 且 `state !== 'idle'`），那么这部空闲电梯就不会去这层。这会导致如果 3 部电梯都在去 5 楼接人（假设它们几乎同时检查且逻辑有漏洞），不过我们之前分析过，`some` 检查会防止多个电梯认领同一层。等等，由于 `tick` 是同步循环 `elevators.forEach`，在 forEach 中，如果电梯0认领了楼层 5（设置 `targetFloor=5, state='moving'`），此时轮到电梯1检查时，`findTargetFloor` 里的 `elevators.some` 会立刻看到电梯0的 `targetFloor === 5 && state !== 'idle'`，于是电梯1就不会认领 5 楼。因此绝对不会有多部电梯奔向同一个等待楼层。这是非常稳健的！

再看一下 `moveTo` 函数：
```javascript
function moveTo(elevator, floor, callback) {
    elevator.targetFloor = floor;
    elevator.state = 'moving';
    ...
}
```
如果是“去接人”，`targetFloor` 是接人楼层，所以会被 `some` 检查到（认领）。当电梯到达接人楼层，在 `doorOpen` 状态的 `setTimeout` 里，如果接到了人，它会调用 `moveTo(elevator, p.targetFloor, ...)`，此时 `elevator.targetFloor` 变成了 `p.targetFloor`（送人楼层）。那么原先接人的楼层就“解除认领”了（因为没有电梯的 `targetFloor` 等于它了）。此时如果还有其他人在那层等，下一 tick 的其他空闲电梯就会发现它未被认领且有人，就会去接。完美！

如果到达接人楼层，但 `queue.length === 0`（比如人突然没了？在我们的程序里，只要没被接走，人就不会少。但如果是被其他电梯接走了？不可能，因为那层被认领了，其他电梯不会来。所以只能是程序自己生成的乘客被接走。所以 `queue.length > 0` 总是成立的，除非有 bug。但为了健壮性，保留了 `else { elevator.state = 'idle'; }` 是对的）。

有一个细节：电梯移动过程中（`moving` 状态），它的 `targetFloor` 是有效的。如果这时有新乘客生成在另一个楼层，且那层没人认领，空闲电梯会去接，不影响正在移动的电梯。这也对。

再看 CSS 动画：
电梯 `transition: top 1s linear, background-color 0.3s;`
每次 `moveTo` 时：
`elevator.element.style.transition = \`top \${duration}s linear, background-color 0.3s\`;`
`elevator.element.style.top = ...`
这很好，速度和距离成正比。

关于“流畅的动画”：电梯上下移动是线性的（linear），很流畅。门开有背景色变化（0.3s）。乘客 tooltip 有 opacity transition。整体比较流畅。

测试边界情况：
如果输入的人数很多（比如 20 人），在同层生成，等待区会塞不下吗？
`.waiting-area` 是 `display: flex; gap: 6px;`。如果不设 `flex-wrap`，会一直往右延伸，可能溢出楼层宽度。
因为楼层有 `flex: 1`，也就是占满剩余宽度，可能比较宽。但以防万一，可以加上 `flex-wrap: wrap;` 或者让等待区可以横向滚动。不过模拟器一般人数不多，或者我们可以加 `flex-wrap: wrap;`。
```css
.waiting-area { display: flex; flex-wrap: wrap; gap: 6px; margin-left: 10px; min-height: 24px; align-content: center; }
```
如果不加 wrap，直接溢出。加上更好。

还有一个点：`building` 高度是 `360px`（6层 * 60px）。电梯 `height: 50px`，井道高度 `360px`。电梯 `top` 变化范围从 `0`（5楼）到 `300px`（0楼）。`300 + 50 = 350px`，正好在 `360px` 的容器内，不超出。对得很齐。

电梯井道背景 `.elevator-shaft`：
有 3 个，每个 `width: 70px`，`left: 0, 70, 140`。`elevatorsContainer` 宽度是 `210px`。正好 3*70=210。
电梯 `.elevator`：`width: 60px`，`left: 5, 75, 145`（`i*70 + 5`）。正好在井道内居中（井道 70，电梯 60，左右各留 5px）。完美。

乘客目标楼层逻辑：
```javascript
let target = Math.floor(Math.random() * NUM_FLOORS);
while (target === floor) {
    target = Math.floor(Math.random() * NUM_FLOORS);
}
```
如果 `NUM_FLOORS` 很小（比如 2），这也能正确生成另一个楼层。这里是 6，肯定没问题。

需求：只给我一个文件。不要查看其他文件，直接创建。提供运行方式和必要的测试说明。
我会在回复里：
1. 给出完整的 HTML 代码（在 ` ```html ` 代码块中）。
2. 给出“运行方式”（比如保存为 `.html` 文件，双击在浏览器打开）。
3. 给出“测试说明”（如何生成乘客，观察电梯依次接送，悬停看 tooltip 等）。

再仔细看需求：
- “每部电梯一次只能搭载一个人” -> 我们的逻辑是电梯有 `passenger` 属性，且接人时只 `shift` 一个，然后立刻去送人，绝不接第二个。满足。
- “一名乘客上电梯后，其他乘客应留在原地等待；当下一部电梯（三部中的任意一部）到达时，他们再依次上电梯。” -> 因为电梯容量1且接完就走，剩下的留在 `floors[f]` 队列，等别的电梯（或自己回来）。满足。
- “每个人都有一个随机分配的目标楼层” -> 满足。
- “鼠标悬停在人物上时，显示一个包含其目标楼层的小提示框” -> 使用 `data-tooltip` 和 CSS `::after`，满足。
- “整体应具有流畅的动画、良好的视觉效果，并保持简洁” -> 使用 flex 布局、阴影、圆角、平滑过渡，满足。
- “全部使用 HTML、CSS 和 JavaScript 构建。只给我一个文件。” -> 单文件，满足。

有没有可能乘客在电梯里时，鼠标悬停 tooltip 被电梯的 `overflow: hidden` 截断？
`.elevator` 没有设置 `overflow: hidden`，所以 tooltip 绝对定位可以正常溢出显示。但 `z-index` 可能需要设置一下，确保 tooltip 在顶层。我们在 `.passenger::after` 里设了 `z-index: 10;`。但电梯本身可能会遮挡别的？同级元素后写的对上层的覆盖。不过由于 tooltip 是在 `bottom: 130%` 往上飘，一般不会被遮挡，除非紧挨着上面的电梯。为了更保险，可以给 `.elevator` 加 `z-index`，或者在 hover 时提升。不过模拟器简单即可，不一定需要极致 z-index 管理。

等等，有一个细节：电梯在移动时，如果带有乘客，乘客的 DOM 是电梯的子元素，所以乘客会跟着电梯平滑移动。这很棒！
如果乘客在等待区，等待区是楼层（从上到下排列）的子元素，不会动。这也很棒！

再看一下代码中的 `FLOOR_HEIGHT = 60;`，而 CSS 里 `.floor` 的 `height: 60px;`。完全对应。如果浏览器有默认 margin，因为我们在 `body` 重置了 `margin: 0` 或用了 `padding`，且 `.building` 是 `display: flex; height: 360px;`，这没问题。
但是 `.floor` 的 `box-sizing` 默认是 `content-box`，如果有 `border-bottom: 1px dashed #eee;`，实际高度会变成 `61px` 吗？如果 `height: 60px` 且 `border-bottom: 1px`，总高度是 61px。6层就是 366px，而 `.building` 高度是 360px，会导致溢出或者压缩？
为了避免这个问题，最好加 `box-sizing: border-box;`，或者在 CSS 开头统一加：
```css
* { box-sizing: border-box; }
```
这样 `.floor` 的 `height: 60px` 就包含了 `border-bottom`，总高度严格 360px。这很重要。
另外 `.elevator` 的 `height: 50px`，加上 `box-sizing: border-box` 也没问题。

还有，`elevatorsContainer` 的 `height` 没有明确写，它是 `.building` 的子元素，`.building` 是 `display: flex; height: 360px;`，那么 `.elevators` 作为 flex 子项，默认 `align-items: stretch`，高度会被拉成 360px。完美。

修改一下 CSS 加上 `box-sizing: border-box;`：
```css
* { box-sizing: border-box; }
```
这样所有高度计算都精确。

再想一想，电梯初次加载在 0 层（最下面），`top: 300px`。由于 `.elevators` 高度 360px，电梯高度 50px，300+50=350，底部留白 10px，看起来正好。如果是 5 层（最上面），`top: 0`，顶部留白 0，电梯占据 0~50，楼层 5 是从 0~60，居中对齐的话，电梯在楼层里偏上一点点（楼层高60，电梯高50，差10，如果楼层内容是垂直居中，电梯 y 从 5 到 55，而我们的 top 是 0 到 50，其实差 5px，都在可视范围内，完全没问题）。

再检查一下“生成乘客”控件：
默认 `value="1"`。最小是 `1`。如果用户故意输入 0 或负数，`parseInt` 可能得到 `0` 或 `NaN`，我用了 `|| 1` 保证至少 1 人。如果是负数输入被 `min="1"` 限制，但用户可能输入 `e`，`parseInt` 会 `NaN`，`NaN || 1` 得到 1。没问题。

还有一个需求：“每部电梯一次只能搭载一个人。总共有三部电梯。一名乘客上电梯后，其他乘客应留在原地等待；当下一部电梯（三部中的任意一部）到达时，他们再依次上电梯。”
现在逻辑是：电梯到达，接一个人，然后立刻去送（因为一次只能搭载一个人）。所以其他乘客确实是在原地等。下一部电梯（或者同一部电梯送完回来）到达时再接下一个。完全吻合。

但我再确认一下：如果有一部电梯到达某层，而其他电梯此时空闲，会怎样？
比如 3 楼有 5 个人。0、1、2 号电梯都在 0 楼 idle。
Tick：0 号认领 3 楼去接，1、2 号没目标 idle。
0 号到达 3 楼，接走 1 人（剩 4 人），去目标楼层（比如 5 楼）。
此时 3 楼未被认领（因为 0 号 target 变成 5 楼）。
下一个 Tick：1 号空闲，认领 3 楼去接。2 号空闲没目标。
1 号到达 3 楼，接走 1 人（剩 3 人），去送。
下一个 Tick：2 号空闲，认领 3 楼去接。...
接着 0 号送完回 idle，发现 3 楼还有人，认领，去接...
所以三部电梯会“依次”或“轮流”来接，完美实现了“依次上电梯，由三部中的任意一部到达时上电梯”。

另外，乘客在等待区排队的顺序是按照生成顺序（`push` 到数组），接人时 `shift`（从队首接走）。这很合理。

关于提示框，需求：“鼠标悬停在人物上时，显示一个包含其目标楼层的小提示框”。我们用了 `.passenger:hover::after`，内容 `目标: X层`。满足。

视觉效果：
背景：浅灰。卡片：白底圆角阴影。
电梯：蓝色，门开变绿。
乘客：红色圆点。
楼层：底部虚线分隔，标签清晰。
整体简洁现代。满足。

最后，代码结构：
```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>...<style>...</style></head>
<body>
...
<script>...</script>
</body>
</html>
```

有没有需要优化的 JS 代码？
`let claimed = elevators.some(e => e !== elevator && e.targetFloor === f && e.state !== 'idle');`
如果电梯认领了楼层 f，但在 `moving` 过程中（即 `state === 'moving'`），这是 `!== 'idle'`，所以会被认为是 claimed。如果到达了楼层 f，进入了 `doorOpen` 状态，此时 `state` 是 `'doorOpen'`，也不是 `'idle'`，所以依然是 claimed，直到它接走人并把 `targetFloor` 改为新目标（或者没人变为 idle）。因此在它开门的 0.6s 内，别的电梯不会来抢这层。这也合理，因为本来就是它来接的。接完人后 `targetFloor` 改变，认领释放。所以非常稳。

有一个边界情况：如果电梯到达楼层 f 且 `queue.length === 0`（比如在极端竞态下？但我们是单线程 JS，不可能有竞态，唯一移除队列是在 `shift`，而只有认领者才会 `shift`。所以认领者到达时，`queue.length` 必定 > 0）。所以 `else` 分支只是防御性编程。

再检查一下动画时长与 setTimeout 是否匹配：
`duration = distance * 1.0;` -> 字符串拼接 `top ${duration}s linear`。
`setTimeout(..., duration * 1000);` -> 完全匹配。
在到达后停留 `600ms` (开门时间)，然后接人、关门、再 `moveTo`。
这部分没有用 transition 做门开关，而是直接用背景色变化（0.3s）。够简单，也够直观。如果想做“门打开”的感觉，可以用伪元素，但没必要增加复杂度。变绿已经很明确。

测试一下 0 层生成人到 5 层：
人生成在 0 层。电梯在 0 层。电梯发现目标 0 层（距离 0）。`duration = 0 * 1.0 = 0s`。
`moveTo` 设置 `top` 一样，`setTimeout` 0ms 后触发回调。
到达，`state = doorOpen`，背景变绿。
600ms 后，接人，`moveTo` 去 5 层，`duration = 5s`。
等等，电梯原来就在 0 层，`top` 不变，`transition: top 0s linear`，立刻触发，没问题。
但是 `setTimeout(..., 0)` 可能会和浏览器的渲染冲突吗？一般没问题。如果怕 `duration=0` 时 transition 不触发或者 setTimeout 太快，可以加一个极小的保底时间，比如 `Math.max(distance, 0.1) * 1000`？其实现代浏览器对于 `setTimeout(0)` 完全没问题，会把它放到下一个宏任务执行，状态机会正常流转。

不过，为了安全，移动时间可以稍微修改：
如果 `distance === 0`，其实不需要移动，但 `moveTo` 依然会走流程。这没问题。

再看一下 `generate-btn` 点击时：
如果同时生成很多人（比如 10 人在同一层），`for` 循环里每次 `floors[floor].push(p)`。此时 `floors[floor].length` 变为 10。认领逻辑会触发一部电梯来接，接走 1 个，然后下一部来接，直到接完。完美。

关于“运行方式和必要的测试说明”：
我会在代码下方写：
### 运行方式
1. 将上述代码保存为 `elevator.html`。
2. 使用任意现代浏览器（如 Chrome、Edge、Firefox）双击打开该文件即可运行。
### 测试说明
1. 在控制面板选择目标楼层和人数，点击“生成乘客”。
2. 观察对应楼层出现红色圆点（乘客），鼠标悬停可查看目标楼层提示框。
3. 电梯（蓝色方块）将自动调度前往接客，到达时变绿（开门），接走一名乘客后（乘客进入电梯内）随即前往目标楼层。
4. 由于每部电梯限乘 1 人，同一楼层多名乘客将依次由空闲电梯接送。
5. 观察多电梯协同与平滑移动动画。

我觉得这个设计已经非常完善了。

检查代码拼写和语法：
- `const NUM_FLOORS = 6;` 和 `const NUM_ELEVATORS = 3;`
- `elevators.push({...})`
- `document.getElementById('floor-select').value` 是字符串，用 `parseInt` 转成数字。
- `document.getElementById('num-people').value`。
- `floorsContainer.appendChild(floorDiv);`
- `getTopForFloor(floor)` 返回数字，在设置 style 时加了 `'px'`：`getTopForFloor(floor) + 'px'`，对的。
- `p.element.setAttribute('data-tooltip', \`目标: \${target}层\`);`
- `floors[floor].push(p);`

一切看起来都很好。

再确认一下电梯在接人之后，去送人的逻辑：
```javascript
// 去送人
moveTo(elevator, p.targetFloor, () => {
    elevator.state = 'doorOpen';
    elevator.element.classList.add('door-open');
    
    setTimeout(() => {
        elevator.element.classList.remove('door-open');
        p.element.remove(); // 乘客离开
        elevator.passenger = null;
        elevator.state = 'idle';
    }, 500);
});
```
到达送人楼层后，停留 500ms（绿状态），然后移除乘客 DOM，电梯变为 idle。下一个 tick 时，它会寻找新的目标（包括可能刚刚离开的楼层如果还有人）。完全正确。

等等，乘客 DOM 被 `p.element.remove()` 后，已经完全从页面移除，不会遗留。由于乘客之前被 `elevator.element.appendChild(p.element)` 移动过，它确实是电梯的子元素，所以从电梯里消失，视觉上就是乘客下电梯了。
如果想要乘客下电梯后消失在楼层，这符合模拟器通常的设定（到达目的地就离开大楼了）。没问题。

有一个小细节：当乘客在电梯里时，电梯高度是 50px，乘客 24px，居中显示。当电梯变绿（开门）时，乘客在绿色背景上显示为红色圆点，对比度可以。如果不明显，可以在 `.elevator.door-open` 时把背景设为较浅的颜色？比如 `#a8e6cf`。其实 `#2ecc71` 是亮绿色，红色圆点在上面是清楚的。

再考虑一个潜在问题：
`setInterval(tick, 200)` 每隔 200ms 运行一次，每次遍历所有电梯。如果电梯状态不是 `idle`，就跳过。这非常高效且安全。不会造成重复派单。

最后检查单文件要求：
需求：“全部使用 HTML、CSS 和 JavaScript 构建。只给我一个文件。不要查看其他文件，直接创建。”
代码全部内联在 `<html>` 中，没有外部引用，满足。

有没有什么遗漏的？
- “每部电梯一次只能搭载一个人” -> 实现了，通过一次只 `shift` 一个，并马上离开。
- “总共有三部电梯” -> 实现了。
- “一名乘客上电梯后，其他乘客应留在原地等待；当下一部电梯（三部中的任意一部）到达时，他们再依次上电梯。” -> 实现了。
- “每个人都有一个随机分配的目标楼层” -> 实现了。
- “鼠标悬停在人物上时，显示一个包含其目标楼层的小提示框” -> 实现了。
- “整体应具有流畅的动画、良好的视觉效果，并保持简洁” -> 实现了。

最后输出格式：
先给出代码（使用 ```html 代码块），然后给出运行方式和测试说明。直接完成任务并交付可运行产物。

可以再给 tooltip 加个箭头吗？没必要，简单的小黑框就够了。
为了更美观，可以给 `body` 设置一点字体平滑：`-webkit-font-smoothing: antialiased;`。
在 CSS 里，`.passenger` 可以加 `transition: transform 0.2s;` 然后 `:hover { transform: scale(1.1); }`，这样 hover 时乘客本身也会有一点动画反馈，增强交互感。虽然需求只说 tooltip，但这让视觉效果更好。

加上：
```css
.passenger { transition: transform 0.2s; }
.passenger:hover { transform: scale(1.15); }
```
很好。

电梯门如果是真正的门（两条缝）会更好看吗？
为了简洁，用纯色背景变绿表示门开是最简洁的。需求说“保持简洁”，所以不再加复杂门动画。

代码总览：
HTML结构清晰，CSS变量少但直观，JS逻辑分层（数据、渲染、调度）。完全符合交付标准。

现在整理成最终回答。
不需要解释太多废话，直接给出文件和说明。
确保 `<select>` 的默认选项和生成逻辑不会出错。楼层 0~5。下拉框里的 `value` 是字符串，如 `"0"`，`parseInt` 转数字。

完成。