# freeCode — Scaffold + Run + Evaluate

Two phases:
1. **Scaffold** — generate a placeholder + exhaustive test cases. User fills in the logic.
2. **Evaluate** — when user sends back their solution, compile and run it, then deliver a full analysis.

The user solves the actual logic; this skill handles everything else.

---

## Workflow

### 1. Collect the Problem

If the user hasn't provided a problem yet, ask:

> "Paste the problem statement (or LeetCode link / problem name) and I'll generate a scaffold with test cases."

If the problem is already in the conversation, proceed immediately — do **not** ask again.

Accept any format:
- Full problem statement (copy-pasted)
- LeetCode / GFG problem name or number
- Brief natural-language description ("two sum but with duplicates")

### 2. Web Search (if problem is named or numbered)

If the user provided a **problem name or number** (e.g., "LC 1976", "Plates Between Candles"), use web search to fetch:
- The full official problem statement
- Constraints section
- Official example inputs/outputs (use these as seed test cases)

Search query pattern: `LeetCode <number or name> problem statement constraints`

If the full statement was pasted directly, skip web search — use what's given.

### 3. Detect Language

Default: **Java 21**

Override if user says: Python, C++, Go, JavaScript, etc.

### 4. Analyze the Problem — Identify All Test Case Categories

Before writing a single line of code, mentally enumerate test cases across these categories (use all that apply):

| Category | Examples |
|---|---|
| **Happy path** | Typical valid inputs with known outputs |
| **Minimum input** | n=0, n=1, empty array/string |
| **Maximum input** | n=10^5, all same elements, very long string |
| **All same elements** | `[5,5,5,5]` |
| **Already sorted** | ascending, descending |
| **Single element** | `[42]` |
| **Two elements** | boundary between trivial and non-trivial |
| **Negative numbers** | if the problem allows |
| **Zeros** | standalone zero or mixed with others |
| **Duplicates** | many repeated values |
| **Target not found** | when answer is -1 / false / empty |
| **Target at boundary** | first index, last index |
| **Overflow-prone** | large sums that exceed int range |
| **Graph-specific** | disconnected graph, self-loop, single node, no edges |
| **Tree-specific** | null root, single node, skewed tree, complete tree |
| **String-specific** | empty string, single char, all same char, mixed case |
| **DP-specific** | all increasing, all decreasing, alternating |

### 5. Generate the Scaffold

#### Java 21 Template (default)

```java
import java.util.*;

class Main {

    // ─────────────────────────────────────────────
    // TODO: implement your solution here
    // ─────────────────────────────────────────────
    static <ReturnType> solve(<Parameters>) {
        // your logic
        return <defaultValue>;
    }

    // ─────────────────────────────────────────────
    // Test Runner
    // ─────────────────────────────────────────────
    static int passed = 0, failed = 0;

    static void check(String label, <ReturnType> expected, <ReturnType> actual) {
        if (Objects.equals(expected, actual)) {
            System.out.println("✅ PASS | " + label);
            passed++;
        } else {
            System.out.println("❌ FAIL | " + label);
            System.out.println("        Expected : " + expected);
            System.out.println("        Actual   : " + actual);
            failed++;
        }
    }

    public static void main(String[] args) {

        // ── Test 1: <category> ──────────────────
        check("Test 1 — <label>",
            <expected>,
            solve(<input>)
        );

        // ── Test 2: <category> ──────────────────
        check("Test 2 — <label>",
            <expected>,
            solve(<input>)
        );

        // ... all test cases ...

        System.out.println("\n─────────────────────────────");
        System.out.println("Results: " + passed + " passed, " + failed + " failed.");
    }
}
```

#### Rules for filling in the template:

1. **Replace** `<ReturnType>`, `<Parameters>`, `<defaultValue>` with the actual types from the problem.
2. **Use `check()`** for every test — no raw `System.out.println` for assertions.
3. **Label every test** with its category: `"Test 3 — All same elements"`, `"Test 7 — Target not found"`.
4. **Hardcode expected values** — compute them by hand or reasoning, not by calling `solve()`.
5. **For array/list return types**, wrap in `Arrays.asList(...)` or `List.of(...)` for `Objects.equals` to work.
6. **For floating point**, use a delta comparison helper instead of `Objects.equals`.
7. **Comment sections** using `// ── Category ──` to visually group related tests.
8. **Minimum 8 test cases**, covering at least 6 distinct categories. More is better.
9. Keep `solve()` as a **stub** — return type correct, body is just `return <defaultValue>`.

---

## Output Format

Do **not** create a file artifact. Instead, render the editor using the `show_widget` (Visualizer) tool — this is the only context where `sendPrompt()` works to send code back to chat.

### Editor Widget Spec

Call `show_widget` with an HTML widget containing:

- A `<textarea>` pre-filled with the full scaffold code
- A **"▶ Submit Solution"** button that calls `sendPrompt(textarea.value)`
- Monospace font, dark background
- Small header showing problem name and language

**Template (HTML — pass to show_widget):**

```html
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/codemirror/5.65.16/codemirror.min.css">
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/codemirror/5.65.16/theme/dracula.min.css">
<style>
*{box-sizing:border-box;margin:0;padding:0}
#er{height:500px;display:flex;flex-direction:column;border-radius:10px;overflow:hidden;border:1px solid #30363d;background:#0d1117;font-family:monospace}
#er:fullscreen{border-radius:0}
.tb{display:flex;align-items:center;gap:6px;padding:6px 10px;background:#161b22;border-bottom:1px solid #30363d;min-height:40px;flex-shrink:0}
.tb select{background:#21262d;color:#c9d1d9;border:1px solid #30363d;border-radius:5px;padding:3px 8px;font-size:12px;cursor:pointer;font-family:monospace}
.sp{flex:1}
.btn{background:transparent;color:#8b949e;border:1px solid #30363d;border-radius:5px;padding:3px 10px;font-size:12px;cursor:pointer;font-family:monospace;white-space:nowrap;transition:background .12s,color .12s}
.btn:hover{background:#21262d;color:#e6edf3}
.btn.on{color:#58a6ff;border-color:#388bfd}
.gbtn{background:#238636!important;color:#fff!important;border-color:#2ea043!important}
.gbtn:hover{background:#2ea043!important}
.cw{flex:1;overflow:hidden;min-height:0}
.CodeMirror{height:100%!important;font-size:13px!important;line-height:1.65!important;font-family:'JetBrains Mono','Fira Code','Cascadia Code','Consolas',monospace!important}
.CodeMirror-scroll{height:100%}
.sb{display:flex;align-items:center;gap:10px;padding:2px 12px;background:#0d1117;border-top:1px solid #21262d;font-size:11px;color:#6e7681;font-family:monospace;min-height:22px;flex-shrink:0}
.sd{opacity:.35}
.dot{font-size:9px}
</style>

<div id="er">
  <div class="tb">
    <span class="dot" style="color:#f85149">●</span>
    <span class="dot" style="color:#d29922">●</span>
    <span class="dot" style="color:#3fb950">●</span>
    <select id="ls" style="margin-left:6px">
      <option value="java">Java</option>
      <option value="python">Python</option>
      <option value="js">JavaScript</option>
      <option value="cpp">C++</option>
    </select>
    <div class="sp"></div>
    <button class="btn on" id="hb">◑ Highlight</button>
    <button class="btn" id="fb" title="Fullscreen (F11)">⛶</button>
    <button class="btn gbtn" id="subbtn">▶ Submit</button>
  </div>
  <div class="cw" id="cw">
    <textarea id="ta">// Write your solution here
class Solution {
    public int solve(int[] nums) {
        
        return 0;
    }
}</textarea>
  </div>
  <div class="sb">
    <span id="cp">Ln 1, Col 1</span>
    <span class="sd">|</span>
    <span id="ll">Java</span>
    <span class="sd">|</span>
    <span>UTF-8</span>
    <span class="sd">|</span>
    <span>Tab: 4 spaces</span>
    <div class="sp"></div>
    <span id="lc">7 lines</span>
  </div>
</div>

<script src="https://cdnjs.cloudflare.com/ajax/libs/codemirror/5.65.16/codemirror.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/codemirror/5.65.16/mode/clike/clike.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/codemirror/5.65.16/mode/python/python.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/codemirror/5.65.16/mode/javascript/javascript.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/codemirror/5.65.16/addon/edit/closebrackets.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/codemirror/5.65.16/addon/edit/matchbrackets.min.js"></script>
<script>
const MODES={java:'text/x-java',python:'python',js:'javascript',cpp:'text/x-c++src'};
const LABELS={java:'Java',python:'Python',js:'JavaScript',cpp:'C++'};
let hlOn=true,lang='java';

const ed=CodeMirror.fromTextArea(document.getElementById('ta'),{
  mode:MODES.java,
  theme:'dracula',
  lineNumbers:true,
  indentUnit:4,
  tabSize:4,
  indentWithTabs:false,
  extraKeys:{'Tab':cm=>cm.execCommand('insertSoftTab'),'Shift-Tab':cm=>cm.execCommand('indentLess')},
  autoCloseBrackets:true,
  matchBrackets:true,
  lineWrapping:false,
});

function upd(){
  const c=ed.getCursor();
  document.getElementById('cp').textContent=`Ln ${c.line+1}, Col ${c.ch+1}`;
  document.getElementById('lc').textContent=`${ed.lineCount()} lines`;
}
ed.on('cursorActivity',upd);
ed.on('change',upd);

document.getElementById('ls').onchange=function(){
  lang=this.value;
  if(hlOn)ed.setOption('mode',MODES[lang]);
  document.getElementById('ll').textContent=LABELS[lang];
};

document.getElementById('hb').onclick=function(){
  hlOn=!hlOn;
  ed.setOption('mode',hlOn?MODES[lang]:null);
  this.classList.toggle('on',hlOn);
  this.textContent=(hlOn?'◑':'○')+' Highlight';
};

const fbtn=document.getElementById('fb');
fbtn.onclick=function(){
  const el=document.getElementById('er');
  if(!document.fullscreenElement){
    el.requestFullscreen&&el.requestFullscreen().then(()=>{fbtn.textContent='⊠';}).catch(()=>{
      el.style.position='fixed';
      el.style.inset='0';
      el.style.height='100vh';
      el.style.width='100vw';
      el.style.zIndex='9999';
      el.style.borderRadius='0';
      fbtn.textContent='⊠';
      ed.refresh();
    });
  } else {
    document.exitFullscreen&&document.exitFullscreen();
  }
};

document.addEventListener('fullscreenchange',()=>{
  if(!document.fullscreenElement){
    fbtn.textContent='⛶';
    const el=document.getElementById('er');
    el.style.position='';el.style.inset='';el.style.height='';
    el.style.width='';el.style.zIndex='';el.style.borderRadius='';
  }
  setTimeout(()=>ed.refresh(),60);
});

document.getElementById('subbtn').onclick=()=>sendPrompt(ed.getValue());

new ResizeObserver(()=>ed.refresh()).observe(document.getElementById('cw'));

document.addEventListener('keydown',e=>{
  if(e.key==='F11'){e.preventDefault();fbtn.click();}
});
</script>
```

Replace `PROBLEM_NAME`, `LANGUAGE`, and `SCAFFOLD_CODE` with actual values. Escape any `<` / `>` / `&` in the scaffold code that would break the HTML attribute.

> ⚠️ Do NOT include solution hints, approach suggestions, or algorithm names anywhere in the scaffold or the editor.

After the widget, add a **Test Case Summary** table in chat:

```
| # | Label | Input | Expected |
|---|-------|-------|----------|
| 1 | Happy path | ... | ... |
| 2 | Empty array | [] | ... |
...
```

---

## Language Variants

### Python 3
```python
import sys
input = sys.stdin.readline

def solve(...):
    pass  # TODO

def check(label, expected, actual):
    if expected == actual:
        print(f"✅ PASS | {label}")
    else:
        print(f"❌ FAIL | {label}")
        print(f"        Expected : {expected}")
        print(f"        Actual   : {actual}")

if __name__ == "__main__":
    check("Test 1 — Happy path", expected, solve(...))
    # ...
```

### C++17
```cpp
#include <bits/stdc++.h>
using namespace std;

// TODO: implement
auto solve(...) {
    return ...;
}

int passed = 0, failed = 0;
template<typename T>
void check(string label, T expected, T actual) {
    if (expected == actual) { cout << "✅ PASS | " << label << "\n"; passed++; }
    else { cout << "❌ FAIL | " << label << "\n"
                << "  Expected: " << expected << "\n"
                << "  Actual:   " << actual   << "\n"; failed++; }
}

int main() {
    check("Test 1 — Happy path", expected, solve(...));
    // ...
    cout << "\nResults: " << passed << " passed, " << failed << " failed.\n";
}
```

---

## Quality Bar

Before outputting, verify:
- [ ] Problem statement and constraints are printed at the top of the code block
- [ ] No hints, approach names, or algorithm suggestions anywhere in output
- [ ] At least 8 test cases
- [ ] At least 6 distinct categories covered
- [ ] Every test has a meaningful label
- [ ] Official examples (from web search or user input) are included as the first test cases
- [ ] Expected values are manually verified (not calls to `solve()`)
- [ ] `solve()` stub compiles as-is (correct return type, no logic)
- [ ] Editor is rendered via `show_widget` (Visualizer), NOT a file artifact — `sendPrompt` only works there
- [ ] Submit button uses `onclick="sendPrompt(document.getElementById('code').value)"`
- [ ] Test summary table is present in chat (outside the artifact)

---

## Phase 2 — Run & Evaluate (when user returns with solution)

Trigger: user sends back code with their `solve()` filled in, or says anything like "here's my solution", "run this", "check my code", "does it pass?".

### Detection

Check if the submitted code has a non-stub `solve()` (i.e., body contains actual logic, not just `return null` / `return 0` / `pass`). If it looks like a stub, ask the user to confirm before running.

### Execution

Write the code to a temp file and compile + run it using `bash_tool`:

**Java:**
```bash
mkdir -p /tmp/freecode_run
cat > /tmp/freecode_run/Main.java << 'EOF'
<user's full code here>
EOF
apt-get install -y default-jdk-headless 2>/dev/null | tail -1
cd /tmp/freecode_run && javac Main.java 2>&1 && java Main 2>&1
```

**Python:**
```bash
cat > /tmp/freecode_run/solution.py << 'EOF'
<user's full code here>
EOF
python3 /tmp/freecode_run/solution.py 2>&1
```

**C++:**
```bash
cat > /tmp/freecode_run/sol.cpp << 'EOF'
<user's full code here>
EOF
cd /tmp/freecode_run && g++ -O2 -o sol sol.cpp 2>&1 && ./sol 2>&1
```

### Compile Error Handling

If compilation fails, show the error output clearly and stop. Do **not** guess at the fix — just show the error and let the user fix it.

### Post-Run Analysis

After a successful run, parse the output (lines starting with `✅ PASS` / `❌ FAIL`) and produce this report:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  Run Results — <Problem Name>
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  ✅ Passed : X / N
  ❌ Failed : Y / N
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Then for each **failed** test, show:
```
❌ Test K — <label>
   Input    : <input>
   Expected : <expected>
   Got      : <actual>
```

Then a brief **Analysis** section (3–6 bullet points max):
- Which test **categories** failed (e.g., "edge cases with empty input", "overflow-prone inputs")
- A pattern in the failures if one exists (e.g., "off-by-one on boundary indices")
- Whether it looks like a logic bug, missing null-check, wrong return type, etc.

> ⚠️ Do NOT suggest the fix or reveal the algorithm. Describe *what* is failing, not *how* to fix it.

If **all tests pass**:
```
🎉 All X tests passed.
```
Add a one-line note on any interesting edge cases the solution handled correctly.

### Complexity Prompt (optional)

After the results, ask:
> "What's the time and space complexity of your solution?" 

Wait for their answer — do not volunteer it.
