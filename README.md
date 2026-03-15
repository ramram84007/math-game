<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Math Maze - 5 Levels</title>
    <style>
        body { font-family: 'Sarabun', sans-serif; background-color: #2c3e50; color: #ecf0f1; display: flex; flex-direction: column; align-items: center; margin: 0; padding: 10px; touch-action: manipulation; }
        h1 { color: #f1c40f; font-size: 1.5rem; margin: 10px 0; }
        .info-panel { display: flex; justify-content: space-between; width: 100%; max-width: 400px; background: rgba(0,0,0,0.3); padding: 10px; border-radius: 8px; margin-bottom: 10px; }
        #game-board { display: grid; grid-template-columns: repeat(10, min(8vw, 35px)); grid-template-rows: repeat(10, min(8vw, 35px)); gap: 1px; background-color: #34495e; padding: 5px; border-radius: 10px; border: 2px solid #7f8c8d; }
        .cell { width: 100%; height: 100%; display: flex; align-items: center; justify-content: center; font-size: min(4vw, 18px); border-radius: 2px; }
        .wall { background-color: #7f8c8d; } .path { background-color: #ecf0f1; } .player { background-color: #3498db; border-radius: 50%; }
        .clue { background-color: #f1c40f; } .exit { background-color: #e74c3c; } .exit.open { background-color: #2ecc71; box-shadow: 0 0 10px #2ecc71; }
        #controls { display: grid; grid-template-columns: repeat(3, 60px); gap: 10px; margin-top: 15px; }
        .btn { background: #34495e; border: 2px solid #ecf0f1; color: white; padding: 15px; border-radius: 10px; cursor: pointer; }
        #modal { display: none; position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.9); z-index: 100; justify-content: center; align-items: center; }
        .content { background: white; color: #333; padding: 20px; border-radius: 15px; width: 80%; max-width: 350px; text-align: center; }
        .opt { width: 100%; padding: 10px; margin: 5px 0; border: 1px solid #3498db; background: #f9f9f9; cursor: pointer; }
    </style>
</head>
<body>
    <h1>Math Maze (ด่าน <span id="lv">1</span>/5)</h1>
    <div class="info-panel">
        <div id="stat">ต้องเก็บ 3 คำใบ้ก่อน!</div>
        <div id="scr">📜: 0 / 3</div>
    </div>
    <div id="game-board"></div>
    <div id="controls">
        <div class="btn" onclick="move(0,-1)">⬆️</div>
        <div class="btn" onclick="move(-1,0)">⬅️</div>
        <div class="btn" onclick="move(1,0)">➡️</div>
        <div class="btn" onclick="move(0,1)">⬇️</div>
    </div>
    <div id="modal"><div class="content"><h3 id="q-head">คำถามคณิตศาสตร์</h3><p id="q-txt"></p><div id="q-opts"></div></div></div>

    <script>
        const levels = [
            [[1,1,1,1,1,1,1,1,1,1],[1,2,0,4,0,1,0,0,0,1],[1,1,1,0,1,1,0,1,0,1],[1,4,0,0,0,0,0,1,0,1],[1,0,1,1,1,1,0,1,0,1],[1,0,1,0,0,0,0,0,0,1],[1,0,1,0,1,1,1,1,0,1],[1,0,0,0,4,0,0,0,0,1],[1,1,1,1,1,1,1,1,3,1],[1,1,1,1,1,1,1,1,1,1]],
            [[1,1,1,1,1,1,1,1,1,1],[1,2,0,0,0,0,0,0,4,1],[1,1,1,1,1,1,1,1,0,1],[1,4,0,0,0,0,0,0,0,1],[1,0,1,1,1,1,1,1,1,1],[1,0,0,0,4,0,0,0,0,1],[1,1,1,1,1,1,1,1,0,1],[1,0,0,0,0,0,0,0,0,1],[1,0,1,1,1,1,1,1,3,1],[1,1,1,1,1,1,1,1,1,1]],
            [[1,1,1,1,1,1,1,1,1,1],[1,2,0,4,0,0,0,0,4,1],[1,0,1,1,1,1,1,1,0,1],[1,0,0,0,0,0,0,0,0,1],[1,1,1,0,1,1,1,1,1,1],[1,0,0,0,4,0,0,0,0,1],[1,0,1,1,1,1,1,1,0,1],[1,0,0,0,0,0,0,0,0,1],[1,1,1,1,1,1,1,1,3,1],[1,1,1,1,1,1,1,1,1,1]],
            [[1,1,1,1,1,1,1,1,1,1],[1,2,0,0,0,0,0,0,0,1],[1,0,1,1,1,1,1,1,0,1],[1,0,4,0,0,0,0,0,0,1],[1,0,1,1,1,1,1,1,0,1],[1,0,0,0,4,0,0,0,0,1],[1,1,1,1,1,1,1,1,0,1],[1,0,4,0,0,0,0,0,0,1],[1,0,1,1,1,1,1,1,3,1],[1,1,1,1,1,1,1,1,1,1]],
            [[1,1,1,1,1,1,1,1,1,1],[1,2,0,0,0,0,0,0,4,1],[1,1,1,1,1,1,1,1,0,1],[1,4,0,0,0,0,0,0,0,1],[1,0,1,1,1,1,1,1,1,1],[1,0,0,0,4,0,0,0,0,1],[1,1,1,1,1,1,1,1,0,1],[1,0,0,0,0,0,0,0,0,1],[1,0,1,1,1,1,1,1,3,1],[1,1,1,1,1,1,1,1,1,1]]
        ];
        
        const qBank = [
            { q: "ถ้า A = {1, 2, 3} และ B = {3, 4, 5} แล้ว A ∩ B คือ?", a: "{3}", opts: ["{1, 2}", "{3}", "{1, 2, 4, 5}", "{1, 2, 3, 4, 5}"] },
            { q: "ประพจน์ 'ถ้า 2 เป็นเลขคู่ แล้ว 2 เป็นจำนวนเฉพาะ' มีค่าความจริงเป็น?", a: "จริง", opts: ["จริง", "เท็จ", "สรุปไม่ได้", "ผิดทุกข้อ"] },
            { q: "นิเสธของ 'ฝนตกและรถติด' คือ?", a: "ฝนไม่ตกหรือรถไม่ติด", opts: ["ฝนไม่ตกและรถไม่ติด", "ฝนตกและรถไม่ติด", "ฝนไม่ตกหรือรถไม่ติด", "ฝนตกหรือรถติด"] },
            { q: "เซตว่าง (∅) เป็นสับเซตของทุกเซตหรือไม่?", a: "ใช่", opts: ["ใช่", "ไม่ใช่", "เฉพาะเซตจำกัด", "แล้วแต่กรณี"] },
            { q: "จำนวนสมาชิกของพาวเวอร์เซต P({a, b}) คือ?", a: "4", opts: ["2", "3", "4", "8"] },
            { q: "ค่าความจริงของ '5 > 3 และ 2 > 4' คือ?", a: "เท็จ", opts: ["จริง", "เท็จ", "สรุปไม่ได้", "ไม่มีค่า"] },
            { q: "ถ้า P เป็นเท็จ แล้ว P ∨ Q จะเป็นจริงเมื่อใด?", a: "Q เป็นจริง", opts: ["Q เป็นจริง", "Q เป็นเท็จ", "เป็นจริงเสมอ", "เป็นเท็จเสมอ"] },
            { q: "เซตของจำนวนเต็มที่อยู่ระหว่าง -2 ถึง 2 คือ?", a: "{-1, 0, 1}", opts: ["{-2, -1, 0, 1, 2}", "{-1, 0, 1}", "{0, 1}", "{-1, 1}"] },
            { q: "สับเซตแท้ของ {1} มีกี่เซต?", a: "1", opts: ["0", "1", "2", "4"] },
            { q: "ประพจน์ p → q สมมูลกับประพจน์ใด?", a: "~q → ~p", opts: ["q → p", "~q → ~p", "~p → ~q", "p ∨ q"] }
        ];

        let lv=0, pos={x:1, y:1}, found=0, modal=false, map=levels[0].map(r=>[...r]);

        function draw() {
            const b = document.getElementById("game-board"); b.innerHTML = '';
            map.forEach((r,y) => r.forEach((c,x) => {
                const d = document.createElement("div"); d.className = "cell";
                if(x==pos.x && y==pos.y) { d.classList.add("player"); d.innerText="🧙‍♂️"; }
                else if(c==1) d.classList.add("wall");
                else if(c==4) { d.classList.add("clue"); d.innerText="📜"; }
                else if(c==3) { d.classList.add("exit"); d.innerText=found>=3?"🚪":"🔒"; if(found>=3) d.classList.add("open"); }
                else d.classList.add("path");
                b.appendChild(d);
            }));
        }

        function move(dx, dy) {
            if(modal) return;
            let nx = pos.x+dx, ny = pos.y+dy;
            if(map[ny][nx] !== 1) {
                pos = {x:nx, y:ny};
                if(map[ny][nx] == 4) showQ();
                else if(map[ny][nx] == 3 && found >= 3) {
                    if(lv<4) { lv++; map=levels[lv].map(r=>[...r]); pos={x:1, y:1}; found=0; document.getElementById("lv").innerText=lv+1; document.getElementById("scr").innerText="📜: 0 / 3"; }
                    else { alert("ยินดีด้วย! คุณเก่งมาก!"); location.reload(); }
                }
                draw();
            }
        }

        function showQ() {
            modal=true; const q = qBank[Math.floor(Math.random()*qBank.length)];
            document.getElementById("q-txt").innerText = q.q;
            const oDiv = document.getElementById("q-opts"); oDiv.innerHTML = '';
            q.opts.forEach(o => {
                const b = document.createElement("button"); b.className="opt"; b.innerText=o;
                b.onclick = () => { if(o==q.a) { map[pos.y][pos.x]=0; found++; document.getElementById("scr").innerText=`📜: ${found} / 3`; modal=false; document.getElementById("modal").style.display="none"; draw(); } else alert("ผิด! ลองใหม่"); };
                oDiv.appendChild(b);
            });
            document.getElementById("modal").style.display="flex";
        }
        draw();
    </script>
</body>
</html>
