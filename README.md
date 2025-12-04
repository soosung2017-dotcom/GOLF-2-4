# GOLF-2-4
<!DOCTYPE html>
<html lang="ko">
<head>
<meta charset="UTF-8" />
<title>골프 머니 게임 계산기</title>
<style>
body {
font-family: sans-serif;
padding: 16px;
}
h1 {
font-size: 20px;
margin-bottom: 8px;
}
table {
border-collapse: collapse;
width: 100%;
max-width: 1000px;
font-size: 12px;
}
th, td {
border: 1px solid #ccc;
padding: 4px;
text-align: center;
}
th {
background: #f0f0f0;
}
input[type="number"] {
width: 50px;
}
.totals {
margin-top: 12px;
max-width: 600px;
}
.totals td, .totals th {
border: 1px solid #ccc;
padding: 4px;
text-align: center;
}
.controls {
margin-bottom: 10px;
}
.note {
font-size: 11px;
color: #555;
margin-top: 8px;
line-height: 1.4;
}
</style>
</head>
<body>
<h1>골프 머니 게임 계산기 (4인 / 타당 2·4 / 버디·이글·스킨 포함)</h1>

<div class="controls">
스킨 값 ($):
<input type="number" id="skinValue" value="2" step="0.5" />
<button onclick="calcAll()">계산하기</button>
</div>

<table id="scoreTable">
<thead>
<tr>
<th>Hole</th>
<th>Par</th>
<th>A 스코어</th>
<th>B 스코어</th>
<th>C 스코어</th>
<th>D 스코어</th>
<th>Rate<br/>($/타)</th>
<th>A Net</th>
<th>B Net</th>
<th>C Net</th>
<th>D Net</th>
<th>이 홀<br/>거래금액</th>
<th>Pot Before<br/>(스킨 개수)</th>
<th>Pot Used</th>
<th>Pot 남은 것</th>
<th>Skin Winner</th>
</tr>
</thead>
<tbody>
<!-- 18홀 생성 -->
<script>
for (let h = 1; h <= 18; h++) {
document.write('<tr>');
document.write('<td>' + h + '</td>');
document.write('<td><input type="number" id="par_' + h + '" value="4" /></td>');
document.write('<td><input type="number" id="A_' + h + '" /></td>');
document.write('<td><input type="number" id="B_' + h + '" /></td>');
document.write('<td><input type="number" id="C_' + h + '" /></td>');
document.write('<td><input type="number" id="D_' + h + '" /></td>');
document.write('<td id="rate_' + h + '"></td>');
document.write('<td id="netA_' + h + '"></td>');
document.write('<td id="netB_' + h + '"></td>');
document.write('<td id="netC_' + h + '"></td>');
document.write('<td id="netD_' + h + '"></td>');
document.write('<td id="moved_' + h + '"></td>');
document.write('<td id="potBefore_' + h + '"></td>');
document.write('<td id="potUsed_' + h + '"></td>');
document.write('<td id="potRemain_' + h + '"></td>');
document.write('<td id="skinWinner_' + h + '"></td>');
document.write('</tr>');
}
</script>
</tbody>
</table>

<table class="totals">
<tr>
<th></th>
<th>A</th>
<th>B</th>
<th>C</th>
<th>D</th>
</tr>
<tr>
<th>전체 합계</th>
<td id="totalA"></td>
<td id="totalB"></td>
<td id="totalC"></td>
<td id="totalD"></td>
</tr>
</table>

<div class="note">
사용법:<br/>
1) 각 홀의 Par, A/B/C/D 스코어를 위에서부터 순서대로 채운 뒤<br/>
2) "계산하기" 버튼을 누르면, 각 홀 Rate(2/4), Net, 스킨, 거래금액, 전체 합계가 자동 계산됩니다.<br/>
&nbsp;&nbsp;- 전 홀에 버디 이상 / 트리플 이상 / 4명 동타 → 다음 홀 타당 $4<br/>
&nbsp;&nbsp;- 버디: 버디한 사람은 각 플레이어에게서 $5씩, 이글: 각 플레이어에게서 $20씩<br/>
&nbsp;&nbsp;- 스킨: 유일한 최저타 &amp; Par 이하일 때만 승리, 이월되며 한 번에 최대 3홀치만 먹기<br/>
</div>

<script>
function calcAll() {
const NUM_HOLES = 18;
const skinValue = parseFloat(document.getElementById('skinValue').value) || 2;

let totalA = 0, totalB = 0, totalC = 0, totalD = 0;

let prevScores = null;
let prevPar = null;
let potRemainPrev = 0;

for (let h = 1; h <= NUM_HOLES; h++) {
const par = parseFloat(document.getElementById('par_' + h).value);
const sA = parseFloat(document.getElementById('A_' + h).value);
const sB = parseFloat(document.getElementById('B_' + h).value);
const sC = parseFloat(document.getElementById('C_' + h).value);
const sD = parseFloat(document.getElementById('D_' + h).value);

const parValid = !isNaN(par);
const scores = [sA, sB, sC, sD];
const allScoresFilled = scores.every(v => !isNaN(v));

// 기본 표시 초기화
document.getElementById('rate_' + h).innerText = '';
document.getElementById('netA_' + h).innerText = '';
document.getElementById('netB_' + h).innerText = '';
document.getElementById('netC_' + h).innerText = '';
document.getElementById('netD_' + h).innerText = '';
document.getElementById('moved_' + h).innerText = '';
document.getElementById('potBefore_' + h).innerText = '';
document.getElementById('potUsed_' + h).innerText = '';
document.getElementById('potRemain_' + h).innerText = '';
document.getElementById('skinWinner_' + h).innerText = '';

// 비어있는 홀부터는 계산 안 함
if (!parValid || !allScoresFilled) {
break;
}

// --- Rate 계산 ---
let rate = 2;
if (h === 1) {
rate = 2;
} else if (prevScores && prevPar != null) {
const hasBirdieOrBetter = prevScores.some(x => x <= prevPar - 1);
const hasTripleOrWorse = prevScores.some(x => x >= prevPar + 3);
const allSame = prevScores.every(x => x === prevScores[0]);
rate = (hasBirdieOrBetter || hasTripleOrWorse || allSame) ? 4 : 2;
}

document.getElementById('rate_' + h).innerText = rate.toFixed(0);

// --- 기본 타수차 Net (pairwise) ---
let net = [0, 0, 0, 0]; // A,B,C,D

for (let i = 0; i < 4; i++) {
for (let j = i + 1; j < 4; j++) {
const diff = (scores[j] - scores[i]) * rate;
net[i] += diff;
net[j] -= diff;
}
}

// --- 버디 보너스 ---
const birdies = scores.map(s => (s === par ? 0 : (s === par - 1 ? 1 : 0)));
// 위 한 줄: 실수 방지용으로 분리할 수도 있음. 여기서는 s === par-1 이면 버디.
const totalBirdies = scores.filter(s => s === par - 1).length;

if (totalBirdies > 0) {
for (let i = 0; i < 4; i++) {
const isBirdie = (scores[i] === par - 1);
if (isBirdie) {
// 버디 한 사람: 버디 안 친 사람 수 × 5
net[i] += (4 - totalBirdies) * 5;
} else {
// 버디 안 한 사람: 버디 수 × -5
net[i] -= totalBirdies * 5;
}
}
}

// --- 이글 보너스 (par -2 이하) ---
const totalEagles = scores.filter(s => s <= par - 2).length;
if (totalEagles > 0) {
for (let i = 0; i < 4; i++) {
const isEagle = (scores[i] <= par - 2);
if (isEagle) {
net[i] += (4 - totalEagles) * 20;
} else {
net[i] -= totalEagles * 20;
}
}
}

// --- 스킨 로직 ---
const potBefore = potRemainPrev + 1;
let potUsed = 0;
let potRemain = potBefore;
let skinWinnerIdx = -1; // 0:A,1:B,2:C,3:D

const minScore = Math.min(...scores);
const minIndices = [];
for (let i = 0; i < 4; i++) {
if (scores[i] === minScore) minIndices.push(i);
}

if (minIndices.length === 1 && minScore <= par) {
// 스킨 승자
skinWinnerIdx = minIndices[0];
potUsed = Math.min(potBefore, 3);
potRemain = potBefore - potUsed;

for (let i = 0; i < 4; i++) {
if (i === skinWinnerIdx) {
net[i] += 3 * potUsed * skinValue; // 3명에게서 받음
} else {
net[i] -= potUsed * skinValue;
}
}
} else {
// 이월
potUsed = 0;
potRemain = potBefore;
}

potRemainPrev = potRemain;

// --- 이 홀 결과 표시 ---
const [nA, nB, nC, nD] = net;
document.getElementById('netA_' + h).innerText = nA.toFixed(0);
document.getElementById('netB_' + h).innerText = nB.toFixed(0);
document.getElementById('netC_' + h).innerText = nC.toFixed(0);
document.getElementById('netD_' + h).innerText = nD.toFixed(0);

// 이 홀에서 움직인 총액 (플러스만 합계)
const moved = [nA, nB, nC, nD].filter(v => v > 0).reduce((a, b) => a + b, 0);
document.getElementById('moved_' + h).innerText = moved.toFixed(0);

document.getElementById('potBefore_' + h).innerText = potBefore.toFixed(0);
document.getElementById('potUsed_' + h).innerText = potUsed.toFixed(0);
document.getElementById('potRemain_' + h).innerText = potRemain.toFixed(0);

const winnerNames = ['A', 'B', 'C', 'D'];
document.getElementById('skinWinner_' + h).innerText =
(skinWinnerIdx >= 0 ? winnerNames[skinWinnerIdx] : '');

// 누적합
totalA += nA;
totalB += nB;
totalC += nC;
totalD += nD;

// 다음 홀 Rate 계산을 위해 저장
prevScores = scores;
prevPar = par;
}

document.getElementById('totalA').innerText = totalA.toFixed(0);
document.getElementById('totalB').innerText = totalB.toFixed(0);
document.getElementById('totalC').innerText = totalC.toFixed(0);
document.getElementById('totalD').innerText = totalD.toFixed(0);
}
</script>
</body>
</html>
