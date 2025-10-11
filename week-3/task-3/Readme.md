# VSDBabySoC — STA Flow (Creative Visual Guide)

*Compact, visual, and actionable documentation for the post-synthesis STA flow used with OpenSTA and sky130 timing libs.*

[![OpenSTA](https://img.shields.io/badge/OpenSTA-v1.0-blue.svg)](https://github.com/OpenSTA/opensta)
[![SkyWater PDK](https://img.shields.io/badge/sky130-PDK-orange.svg)](https://github.com/google/skywater-pdk)
[![Docker](https://img.shields.io/badge/Docker-run-blue.svg)](https://www.docker.com/)

---

## What’s in this guide
- Visual flowchart of the whole STA pipeline
- Improved, easy-to-read PVT corner table
- Graph placeholders and instructions to generate plots (WNS/TNS/Slack)
- Beautified STA Tcl example and plotting script
- Quick troubleshooting & tips (with images/screenshots placeholders)

---

## 1) Visual STA Flow (Mermaid)
A high-level flowchart showing the STA steps from libs → reports.

```mermaid
flowchart TD
  A[Start: Prepare timing libs & netlist] --> B[read_liberty + read_verilog]
  B --> C[link_design vsdbabysoc]
  C --> D[read_sdc]
  D --> E[check_setup]
  E --> F{Generate Reports}
  F --> F1[report_checks (min_max)]
  F --> F2[report_worst_slack -max/-min]
  F --> F3[report_tns / report_wns]
  F --> G[Save to STA_OUTPUT/]
  G --> H[Post-process/Plot (Python)]
  H --> I[Investigate & Fix]
```

---

## 2) PVT Corner Matrix (table)
A concise table that maps each LIB file to the corner type and whether it is setup/hold critical.

| # | Liberty File (example)                                 | Corner Name                  | Timing Type |
|---:|-------------------------------------------------------|-----------------------------|-------------:|
| 1  | sky130_fd_sc_hd__tt_025C_1v80.lib                    | tt_25C_1.8v                 | Nominal     |
| 2  | sky130_fd_sc_hd__ff_100C_1v65.lib                    | ff_100C_1.65v               | Hold (fast) |
| 3  | sky130_fd_sc_hd__ss_100C_1v40.lib                    | ss_100C_1.40v               | Setup (slow)|
| ...| (list continues for all 13 libraries)                 |                             |             |

Tip: color-code the table cells in rendered docs for fast scanning (setup = red, hold = green, nominal = gray).

---

## 3) Improved sta_across_pvt.tcl (example)
Polished Tcl script snippet with better file management, timestamped output, and consistent naming.

```tcl
# sta_across_pvt.tcl (cut down for readability)
set LIB_DIR "/data/VLSI/VSDBabySoC/OpenSTA/examples/timing_libs"
set NETLIST "/data/VLSI/VSDBabySoC/OpenSTA/examples/BabySOC/vsdbabysoc.synth.v"
set SDC "/data/VLSI/VSDBabySoC/OpenSTA/examples/BabySOC/vsdbabysoc_synthesis.sdc"
set OUT_DIR "/data/VLSI/VSDBabySoC/OpenSTA/examples/BabySOC/STA_OUTPUT"
file mkdir $OUT_DIR

set list_of_lib_files(1) "sky130_fd_sc_hd__tt_025C_1v80.lib"
# ... (rest unchanged) ...

for {set i 1} {$i <= [array size list_of_lib_files]} {incr i} {
    set libfile [file join $LIB_DIR $list_of_lib_files($i)]
    read_liberty $libfile
    read_verilog $NETLIST
    link_design vsdbabysoc
    current_design
    read_sdc $SDC

    check_setup -verbose

    set base [file tail $libfile]
    report_checks -path_delay min_max -fields {nets cap slew input_pins fanout} -digits {4} \
        > [file join $OUT_DIR "min_max_${base}.txt"]

    exec echo "$base" >> [file join $OUT_DIR "sta_worst_max_slack.txt"]
    report_worst_slack -max -digits {4} >> [file join $OUT_DIR "sta_worst_max_slack.txt"]

    exec echo "$base" >> [file join $OUT_DIR "sta_worst_min_slack.txt"]
    report_worst_slack -min -digits {4} >> [file join $OUT_DIR "sta_worst_min_slack.txt"]

    exec echo "$base" >> [file join $OUT_DIR "sta_tns.txt"]
    report_tns -digits {4} >> [file join $OUT_DIR "sta_tns.txt"]

    exec echo "$base" >> [file join $OUT_DIR "sta_wns.txt"]
    report_wns -digits {4} >> [file join $OUT_DIR "sta_wns.txt"]

    # cleanup for next corner
    remove_design
}
```

---

## 4) Output structure (what the run creates)
Suggested output directory after a run:

- STA_OUTPUT/
  - min_max_<lib>.txt   (detailed per-corner min/max path report)
  - sta_worst_max_slack.txt  (WNS per corner)
  - sta_worst_min_slack.txt  (Hold slack per corner)
  - sta_tns.txt        (Total Negative Slack per corner)
  - sta_wns.txt        (Worst Negative Slack per corner)
  - plots/             (PNG/SVG plots we will generate)

---

## 5) Visual reports — Graphs & How to generate them

Placeholders for images (already used in prior docs). Generate these plots using the provided Python script (below). The README will reference the images once produced.

- Images used in this document:
  - Images/Worst_Hold_Slack.png
  - Images/Worst_Setup_Slack.png
  - Images/WNS.png
  - Images/TNS.png
  - Images/STA_All_Metrics.png
  - Images/Task3_vsd_babay_slack.png  (example screenshot)

Example: include the images in markdown like this:

![Worst Setup Slack](Images/Worst_Setup_Slack.png)
![WNS across PVT corners](Images/WNS.png)

---

## 6) Python plotting script (example)
Save the script as `tools/plot_sta_results.py`. It reads the txt summary files and creates the graphs.

```python
# tools/plot_sta_results.py
# Usage: python3 tools/plot_sta_results.py --input-dir path/to/STA_OUTPUT --out-dir path/to/STA_OUTPUT/plots
import os, argparse
import matplotlib.pyplot as plt

def read_key_values(filepath):
    # expects lines: "<libname>" followed by numeric values in same order as corners
    keys = []
    vals = []
    with open(filepath) as f:
        for line in f:
            s=line.strip()
            if not s: continue
            # try parse last token as float
            parts=s.split()
            try:
                v=float(parts[-1])
                vals.append(v)
                keys.append(" ".join(parts[:-1]) if len(parts)>1 else parts[0])
            except:
                # if not numeric, just store label
                keys.append(s)
    return keys, vals

if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument("--input-dir", required=True)
    parser.add_argument("--out-dir", required=True)
    args = parser.parse_args()
    os.makedirs(args.out_dir, exist_ok=True)

    # Example files
    files = {
      "wns": "sta_wns.txt",
      "tns": "sta_tns.txt",
      "worst_max": "sta_worst_max_slack.txt",
      "worst_min": "sta_worst_min_slack.txt"
    }

    for name,f in files.items():
        path = os.path.join(args.input_dir, f)
        if not os.path.exists(path):
            print("Missing", path); continue
        keys, vals = read_key_values(path)
        plt.figure(figsize=(10,4))
        plt.bar(range(len(vals)), vals, color='tab:blue')
        plt.xticks(range(len(vals)), keys, rotation=45, ha='right', fontsize=8)
        plt.ylabel(name.upper())
        plt.title(f"{name.upper()} across PVT corners")
        plt.tight_layout()
        out = os.path.join(args.out_dir, f"{name}.png")
        plt.savefig(out, dpi=150)
        print("Saved", out)
```

Run example:

```bash
python3 tools/plot_sta_results.py --input-dir /data/VLSI/VSDBabySoC/OpenSTA/examples/BabySOC/STA_OUTPUT \
                                 --out-dir /data/VLSI/VSDBabySoC/OpenSTA/examples/BabySOC/STA_OUTPUT/plots
```

---

## 7) Example Visuals (placeholders)
- Combined STA metrics dashboard  
  ![STA Dashboard](Images/STA_All_Metrics.png)

- WNS (Worst Negative Slack) bar chart  
  ![WNS](Images/WNS.png)

- TNS (Total Negative Slack) bar chart  
  ![TNS](Images/TNS.png)

- Screenshot showing the OpenSTA run (example):  
  ![OpenSTA run screenshot](Images/Task3_vsd_babay_slack.png)

If you want, I can generate example SVG/PNG plots and commit them into the repo (you'll need to provide the raw numeric outputs or let me run scripts on your CI).

---

## 8) Troubleshooting & Quick Tips (with images)
1. Syntax/parsing errors in liberty files:
   - Symptom: OpenSTA stops with "parsing error" (shown below).
   - Fix: Search for unclosed braces/brackets or invalid characters.
   - Example screenshot:  
     ![Parsing Error Example](Images/Parser_Error_Example.png)

2. Large negative WNS:
   - Run `report_checks -path_delay min_max -verbose` and inspect top 5 failing paths.
   - Look for: high net capacitance, large fanout, or a missing timing constraint.

3. Hold violations:
   - Usually show up in fastest corners (ff_... high V, low temp).
   - Remedies: add hold fixes (buffers or gate sizing) or adjust constraints.

---

## 9) Next steps & automation ideas
- Add a GitHub Action that:
  - Runs OpenSTA in Docker for each PR
  - Generates WNS/TNS plots and uploads them as artifacts
  - Compares WNS/TNS delta to baseline and comments on PR
- Create a small HTML dashboard (plotly) that visualizes corner-by-corner metrics interactively.
- Add per-path traceability: link a failing path back to the netlist line & source register.

---

## 10) Call to action
Tell me which visual you want first:
- [ ] I want the WNS/TNS plotted and committed (I will add numeric files)
- [ ] Create GitHub Action to run this automatically
- [ ] Generate annotated failing-path screenshots

If you want, I can create the plotting PNGs and add them to Images/ (or push a PR) — say the word and provide the numeric outputs or allow CI to run OpenSTA.

---

Credits: Adapted from the original OpenSTA BabySoC examples. Visual assets are placeholders; I can generate and commit real images once you provide the raw outputs or enable CI to run the flow.
