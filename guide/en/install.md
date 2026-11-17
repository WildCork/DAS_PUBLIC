[한국어](../install.md) | **English**

# Installation and First Run

DAS needs no installer. Unzip it and run `DAS.exe`.

## 1. Download

Download the latest `DAS_Release_<date>.zip` from the [Releases page](https://github.com/WildCork/DAS_PUBLIC/releases/latest) of this repository.

## 2. Unzip

Unzip it into any folder (for example `C:\DAS`). You get the following:

| Folder / file | Contents |
|---|---|
| `Run\` | `DAS.exe` and the Qt runtime libraries (DLLs) it needs |
| `Data\` | Example configuration (`endpoints.json`), IDL files, example captures (`launch_result_*.pcapng`), example report templates (English `launch_procedure_report_template_example_en.json`, Korean `발사절차_성적서_양식_예시.json`) |
| `README.md` | Program overview |
| `LICENSE.txt`, `ThirdPartyLicenses\` | Terms of use and the licenses of bundled libraries |

- Keep the files in `Run\` together. Copying only `DAS.exe` somewhere else will not run.
- For a desktop shortcut, use **Create shortcut** instead of copying `DAS.exe`.

## 3. Run

Double-click `Run\DAS.exe`.

- If Windows shows a "Windows protected your PC" warning, click **More info → Run anyway**.
- On first start, the empty ① Devices / IDL tab opens. You can switch the UI language (한국어/English) and theme while the program runs.
- Press `F1` on any tab to open that tab's help.

## 4. Open the example data

1. On ① Devices / IDL, press the green **Load Test Analysis Config** button and choose `Data\endpoints.json`. Devices, IDL files and links are filled in at once.
2. Go to ③ Test Analysis Settings, pick `Data\launch_result_1k.pcapng` with **Browse...**, and press **Load**.
3. Check the messages to view in the message tree on the right and press the green **Apply Filter**.
4. Review the results in the Timeline, Periodicity, Continuity and Responsiveness tabs of ④ Visualize.
5. Build and save a report or dashboard in ⑤ Analysis Results.

You can also open a configuration and a capture together from the command line (run from the `Run\` folder):

```powershell
.\DAS.exe --config ..\Data\endpoints.json --pcapng ..\Data\launch_result_1k.pcapng
```

`launch_result_10k.pcapng` is a longer capture of the same scenario; the `_withNoise` and `_anomaly` captures mix in noise and abnormal situations. The example data is a simulated launch procedure, not real test data. To try a report, load `Data\launch_procedure_report_template_example_en.json` from **Manage Template** on ⑤ Analysis Results → Report -- DAS never translates text inside configuration, IDL or template files, so use the English template for English check items.

## 5. Where data is stored, and uninstalling

- Program settings and work sessions are stored under `%LOCALAPPDATA%\DAS`. Nothing is written into the folder you unzipped.
- To uninstall, delete the unzipped folder; to remove settings too, also delete `%LOCALAPPDATA%\DAS`.

## System requirements

- Windows 10/11 64-bit
- 1280×720 or larger display recommended
- Memory depends on capture size (roughly 0.1 GB per million packets + 0.5 GB).
