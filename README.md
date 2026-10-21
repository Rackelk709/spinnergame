# spinnergame<iframe
 
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Spin to Learn | Question Wheel</title>

<style>
* { box-sizing: border-box; }

body {
  margin: 0;
  background: #f5f7fb;
  font-family: Arial, sans-serif;
  color: #18334f;
}

.container {
  max-width: 760px;
  margin: 20px auto;
  padding: 25px;
  background: white;
  border-radius: 16px;
  text-align: center;
  box-shadow: 0 4px 18px #00000015;
}

h1 { color: #c8102e; margin-bottom: 8px; }

.instructions {
  color: #536477;
  line-height: 1.5;
}

.wheel-box {
  position: relative;
  width: 100%;
  max-width: 420px;
  margin: 24px auto;
}

canvas {
  width: 100%;
  height: auto;
  display: block;
}

.pointer {
  position: absolute;
  top: 0;
  left: 50%;
  transform: translateX(-50%);
  width: 0;
  height: 0;
  border-left: 16px solid transparent;
  border-right: 16px solid transparent;
  border-top: 32px solid #172d46;
  z-index: 2;
}

button {
  border: 0;
  padding: 13px 24px;
  margin: 5px;
  border-radius: 9px;
  font-size: 16px;
  font-weight: bold;
  cursor: pointer;
}

button:focus-visible, input:focus-visible {
  outline: 3px solid #2474b5;
  outline-offset: 3px;
}

button:disabled {
  opacity: .5;
  cursor: not-allowed;
}

#spin {
  background: #c8102e;
  color: white;
}

#reset {
  background: #e5ebf2;
  color: #17365b;
}

.result {
  margin-top: 24px;
  padding: 20px;
  background: #f0f4fa;
  border-left: 5px solid #c8102e;
  border-radius: 8px;
  text-align: left;
  line-height: 1.6;
}

.result h2 {
  margin: 0 0 8px;
  font-size: 19px;
}

.result p { margin: 0; }

.option {
  display: block;
  margin-top: 12px;
}

#status {
  font-size: 14px;
  color: #536477;
}

@media (max-width: 500px) {
  .container { margin: 8px; padding: 14px; }
}
</style>
</head>

<body>
<main class="container">
  <h1>Spin to Learn!</h1>
  <p class="instructions">
    Click the button to spin the wheel and
    receive a random reflection question.
    Use course concepts and evidence in your response.
  </p>

  <div class="wheel-box">
    <div class="pointer" aria-hidden="true"></div>
    <canvas id="wheel" width="600" height="600"
      role="img"
      aria-label="Wheel with numbered question sections">
    </canvas>
  </div>

  <button id="spin">Spin the Wheel!</button>
  <button id="reset">Reset</button>

  <label class="option">
    <input type="checkbox" id="noRepeat" checked>
    Avoid repeating questions
  </label>

  <p id="status" aria-live="polite"></p>

  <div class="result" role="status" aria-live="polite">
    <h2 id="resultTitle">Your Question</h2>
    <p id="resultText">
      Spin the wheel to reveal your question!
    </p>
  </div>
</main>

<script>
// =======================================
// INSTRUCTOR EDITING AREA
// Change, add, or remove questions here.
// Keep quotation marks and commas.
// =======================================

const questions = [
  "What is one important concept you learned in this module?",
  "How would you apply this concept to a real-world situation?",
  "What assumption about this topic would you challenge?",
  "What question do you still have about the material?",
  "How might someone with a different perspective view this issue?",
  "What evidence supports an important claim from this module?",
  "How does this topic connect to something you previously learned?",
  "What would you change about a decision made in this scenario?"
];

// =======================================
// SPINNER FUNCTIONALITY
// =======================================

const canvas = document.getElementById("wheel");
const ctx = canvas.getContext("2d");
const spinButton = document.getElementById("spin");
const resetButton = document.getElementById("reset");
const noRepeat = document.getElementById("noRepeat");
const resultTitle = document.getElementById("resultTitle");
const resultText = document.getElementById("resultText");
const statusText = document.getElementById("status");

const colors = [
  "#c8102e", "#17365b",
  "#d83c55", "#28608d",
  "#a70d29", "#407ca5",
  "#b72540", "#24496e"
];

const TWO_PI = Math.PI * 2;
let rotation = 0;
let used = new Set();
let spinning = false;

function drawWheel() {
  const center = canvas.width / 2;
  const radius = center - 15;
  const count = questions.length;

  ctx.clearRect(0, 0, canvas.width, canvas.height);
  if (!count) return;

  const slice = TWO_PI / count;

  for (let i = 0; i < count; i++) {
    const start = rotation - Math.PI / 2
      - slice / 2 + i * slice;
    const end = start + slice;

    ctx.beginPath();
    ctx.moveTo(center, center);
    ctx.arc(center, center, radius, start, end);
    ctx.closePath();

    ctx.fillStyle =
      noRepeat.checked && used.has(i)
      ? "#aab2be"
      : colors[i % colors.length];

    ctx.fill();
    ctx.strokeStyle = "white";
    ctx.lineWidth = 3;
    ctx.stroke();

    const middle = start + slice / 2;
    const x = center + Math.cos(middle) * radius * .68;
    const y = center + Math.sin(middle) * radius * .68;

    ctx.fillStyle = "white";
    ctx.font = "bold 25px Arial";
    ctx.textAlign = "center";
    ctx.textBaseline = "middle";
    ctx.fillText(i + 1, x, y);
  }

  ctx.beginPath();
  ctx.arc(center, center, 36, 0, TWO_PI);
  ctx.fillStyle = "white";
  ctx.fill();

  ctx.fillStyle = "#17365b";
  ctx.font = "bold 15px Arial";
  ctx.textAlign = "center";
  ctx.fillText("SPIN", center, center + 5);
}

function updateStatus() {
  statusText.textContent = noRepeat.checked
    ? used.size + " of " + questions.length
      + " questions completed"
    : "Repeats allowed";
}

function easeOut(t) {
  return 1 - Math.pow(1 - t, 3);
}

function spinWheel() {
  if (spinning || !questions.length) return;

  const available = questions
    .map((_, i) => i)
    .filter(i => !noRepeat.checked || !used.has(i));

  if (!available.length) {
    resultTitle.textContent = "All Questions Complete!";
    resultText.textContent =
      "Click Reset to start a new round.";
    return;
  }

  const selected = available[
    Math.floor(Math.random() * available.length)
  ];

  const slice = TWO_PI / questions.length;
  const normalized = ((rotation % TWO_PI) + TWO_PI) % TWO_PI;
  const target = ((-selected * slice) % TWO_PI + TWO_PI) % TWO_PI;
  const extra = (target - normalized + TWO_PI) % TWO_PI;
  const startRotation = rotation;
  const endRotation = rotation + TWO_PI * 5 + extra;

  const reducedMotion = window.matchMedia(
    "(prefers-reduced-motion: reduce)"
  ).matches;
  const duration = reducedMotion ? 100 : 3500;

  spinning = true;
  spinButton.disabled = true;
  resetButton.disabled = true;
  resultTitle.textContent = "Spinning...";
  resultText.textContent = "Selecting your question.";

  const startTime = performance.now();

  function animate(now) {
    const progress = Math.min(
      (now - startTime) / duration, 1
    );

    rotation = startRotation +
      (endRotation - startRotation) * easeOut(progress);

    drawWheel();

    if (progress < 1) {
      requestAnimationFrame(animate);
    } else {
      rotation = endRotation;
      spinning = false;
      used.add(selected);

      resultTitle.textContent =
        "Question " + (selected + 1);
      resultText.textContent = questions[selected];

      resetButton.disabled = false;
      spinButton.disabled = false;

      if (noRepeat.checked &&
          used.size === questions.length) {
        spinButton.disabled = true;
        spinButton.textContent = "All Questions Complete";
      }

      updateStatus();
      drawWheel();
    }
  }

  requestAnimationFrame(animate);
}

spinButton.addEventListener("click", spinWheel);

resetButton.addEventListener("click", function() {
  if (spinning) return;
  used.clear();
  rotation = 0;
  spinButton.disabled = !questions.length;
  spinButton.textContent = "Spin the Wheel!";
  resultTitle.textContent = "Your Question";
  resultText.textContent =
    "Spin the wheel to reveal your question!";
  updateStatus();
  drawWheel();
});

noRepeat.addEventListener("change", function() {
  spinButton.disabled = !questions.length ||
    (noRepeat.checked && used.size === questions.length);
  spinButton.textContent = spinButton.disabled && questions.length
    ? "All Questions Complete"
    : "Spin the Wheel!";
  updateStatus();
  drawWheel();
});

if (!questions.length) {
  spinButton.disabled = true;
  statusText.textContent = "Add questions in the code.";
} else {
  drawWheel();
  updateStatus();
}
</script>
</body>
</html>

