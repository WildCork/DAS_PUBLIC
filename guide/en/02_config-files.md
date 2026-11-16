[한국어](../02_config-files.md) | **English**

# Step 0: Test Config File Setup

DAS itself is a generic interpreter that doesn't know in advance which launch vehicle or equipment names apply to your test -- telling the program "which equipment exists in this test and what IP:Port it uses," "which IDL should interpret the traffic on a given socket pair," and "how to split that payload's bytes into fields" is exactly what the three kinds of text files covered in this section do. Without any of this information, even a successfully loaded PCAPNG shows every device as an "unidentified IP" and every message as raw bytes -- the steps ① and ② that follow are, in the end, the process of entering what these three files contain directly on screen.

**How to use**

1. Prepare the IDL files first -- they define the structure of the payload each device exchanges (message ID, fields, periodic or not); without them, the other two files have nothing to point to.
2. Once the IDLs are ready, use endpoints.lua to tie together the devices/sockets and which IDL each socket pair uses, all in one table -- if you repeat a test with the same configuration, you only need to load this one file.
3. As an alternative to endpoints.lua, or if you want to keep exactly what you registered on screen, save it as a DAS config JSON -- since DAS itself creates and reads this format, it's normally produced by registering things on screen and then using the save function, rather than being hand-written from the start.
4. For all three files, the sub-items below let you check the required rules (parsing fails or the feature is disabled without them) separately from the optional rules (defaults apply without them).

**Notes**

- None of the three files is mandatory -- registering devices, sockets, links, and IDLs one by one on the ① and ② step screens produces exactly the same result; config files are simply a shortcut that prepares that same input as text in advance so it can be reproduced in one "Load Config."
- endpoints.lua and IDL are meant to be hand-written by a person in a text editor, whereas DAS config JSON is a format the program itself handles directly through its save/load function -- they serve different purposes. If your tests repeat with the same device configuration each time, the endpoints.lua+IDL combination fits naturally; if you want to pick up exactly where you left off after registering things on screen, using save/load config (JSON) is the natural choice.

## Writing Rules for endpoints.lua

The devices table defines each device's sockets (IP:Port), and the protos table specifies which socket pairs communicate with each other and which IDL name should interpret that traffic -- this single file entirely replaces step ①'s device/socket list and step ②'s link and IDL assignment. Note, though, that it isn't a full Lua language interpreter but a dedicated parser that only recognizes the simple return { devices = {...}, protos = {...} } table shape, so using real Lua syntax such as conditionals or functions will fail to parse from the start.

**How to use**

1. [Required] The whole file must be wrapped in a single top-level table, return { devices = { ... }, protos = { ... } } -- if this shape itself can't be found, the file isn't read at all.

   ```
   return {
       devices = { ... },   -- endpoint (device / socket) definitions
       protos  = { ... },   -- socket pair ↔ IDL mapping
   }
   ```

2. [Required] Each protos entry must list two or more socket keys that are actually defined in devices -- an entry with a key not found in devices, or with only one valid key, is skipped entirely and produces no links at all for that protocol.

   ```
   protos = {
       WCU_CMS = { "WCU_CMS_UDP", "CMS_1_UDP" },  -- 2 entries: OK, creates 2 links
       MCU_MS  = { "MCU_MS_UDP",  "MS_1_UDP"  },  -- 2 entries: OK
   },
   ```

3. [Required] The same source-to-destination socket pair must not be assigned different IDLs by different protos entries -- this is the one case that actually raises an Error in the diagnostics dialog, since the program can't decide which IDL should decode it.

   ```
   protos = {
       WCU_CMS  = { "WCU_CMS_UDP", "CMS_1_UDP" },  -- uses WCU_CMS.idl
       WCU_CMS2 = { "WCU_CMS_UDP", "CMS_1_UDP" },  -- same socket pair, but
                                                   -- uses WCU_CMS2.idl → error!
   },
   ```

4. [Effectively required] A device that uses multiple sockets (e.g., one with separate status-report and command ports) must keep the same name prefix across its devices entries -- device grouping only looks at the part of name before the first underscore (_), so a mismatched prefix splits what should be one device into separate devices.

   ```
   WCU_CMS_UDP = { name = "WCU_CMS", ip = "10.3.24.10", port = 30150, type = "UDP" },
   WCU_WCP_UDP = { name = "WCU_WCP", ip = "10.3.24.10", port = 30630, type = "UDP" },
   WCU_MIE_UDP = { name = "WCU_MIE", ip = "10.3.24.10", port = 4300,  type = "UDP" },
   -- everything before the first underscore (_) is "WCU" → merged into one device "WCU" with 3 sockets
   ```


**Notes**

- [Optional] If devices.name is empty, the table key itself is used as the name; if ip is empty, it's registered as an empty string that matches no real traffic; if port is missing, it's treated as 0 ("all ports allowed"); and if type is missing or not UDP/TCP, it's left as protocol unknown -- none of the four prevents the file from being read, but all four directly affect real-world matching accuracy.

  ```
  CMS_1_UDP = { name = "CMS_1", ip = "10.3.24.100", port = 30150, type = "UDP" },
  --            ^^^^^^^^^^^^^^   ^^^^^^^^^^^^^^^^^^^   ^^^^^^^^^   ^^^^^^^^^^^
  --            name (key if empty) IP (no match if empty)  port (0 if empty)  protocol (unknown if empty)
  ```

- [Optional] A .idl file with the same name as the protos key (e.g., protos.WCU_CMS <-> WCU_CMS.idl in the same folder) is linked automatically -- even if that .idl file isn't in the folder, the link itself is still created; that protocol's messages are simply shown as raw bytes with no field interpretation.

  ```
  protos = {
      WCU_CMS = { "WCU_CMS_UDP", "CMS_1_UDP" },
  },
  -- looks for "WCU_CMS.idl" in the same folder as endpoints.lua and links it automatically.
  ```

- If two socket entries point to the same IP:Port (often due to a typo), only a warning appears and the file keeps loading -- it's worth checking the diagnostics dialog carefully so this isn't missed.

  ```
  A_UDP = { name = "A", ip = "10.3.24.10", port = 30150, type = "UDP" },
  B_UDP = { name = "B", ip = "10.3.24.10", port = 30150, type = "UDP" },
  -- A_UDP and B_UDP share the same IP:Port → warning ("check whether this is a typo")
  ```


## DAS Config JSON Format

The save format DAS itself reads and writes, produced by "Save Settings" in step ③ — rather than writing it by hand from scratch, the intended use is to register devices, links, IDLs, and request-response relationships on screen, create the file with the save feature, and later restore it exactly with "Load Settings". The PCAPNG packets themselves and the step ⑤ report check items are not part of this file.

**How to use**

1. [Required] The top level must carry a numeric "version" field of 1 or higher — this is the only condition this file rejects as a hard error. If it is missing or malformed, loading fails outright with an error asking you to confirm that this really is a JSON settings file saved by DAS. The current version writes 2 (which includes request-response relationships); older files saved as 1 still load, simply without any relationships.

   ```
   {
     "version": 2,
     ...
   }
   ```

2. Beyond that, the individual fields of the devices / idlFiles / links / responseRelationships arrays only need to be well-formed, and a file saved normally satisfies this automatically — editing this file by hand is not recommended.

   ```
   {
     "devices":  [ { "id": 1, "name": "WCU", "sockets": [ ... ] } ],
     "idlFiles": [ { "id": 1, "name": "WCU_CMS",
                      "path": "C:/DAS/Config/WCU_CMS.idl" } ],
     "links":    [ { "id": 1, "srcDevId": 1, "dstDevId": 2,
                      "idlId": 1, ... } ]
   }
   ```

3. The request-response relationships defined in ③ Test Analysis Settings (the same definitions ④ Visualize > Responsiveness reads) are stored in the "responseRelationships" array — each element holds the relationship id and color plus request/response objects pointing at the two messages (linkId, idlId, msgId, structName, variantKey, srcDevId, dstDevId, and labelCache for display). A relationship has no separate "name" — the display name (labelCache) is composed automatically from the request→response message path.

   ```
   "responseRelationships": [
     { "id": 1, "color": "#4f9df8",
       "request":  { "linkId": 12, "idlId": 4, "msgId": 3,
                      "structName": "MIE_MCU_HATCH_STATUS", ... },
       "response": { "linkId": 18, "idlId": 7, "msgId": 9,
                      "structName": "WCU_WCP_STATUS", ... },
       "options": {} }
   ]
   ```


**Notes**

- [Optional] Individual fields in devices/idlFiles/links (name, IP:Port, path, etc.) are simply filled with an empty value or 0 if blank or missing, and the file keeps loading regardless. Each idlFiles entry records the IDL's absolute path (path), its path relative to the config file (relativePath), and a content hash at save time (sha256); the top-level idlPathMode (relative or absolute) decides which path is tried first when loading -- if no file is there, the file at the other path is used only when its hash matches. If neither path has it, a file with the same name is looked for in the config file's folder and its subfolders, and you are asked first if the hash differs or is missing.

  ```
  "idlPathMode": "relative",
  "idlFiles": [
    { "id": 1, "name": "WCU_CMS", "path": "C:/DAS/Config/WCU_CMS.idl",
      "relativePath": "WCU_CMS.idl", "sha256": "…" }
  ]
  ```

- For backward compatibility, the older format without socket info (an addresses array with just IP, no port) can still be read, but saving again always rewrites it in the current socket format.

  ```
  "devices": [
    { "id": 1, "name": "WCU", "addresses": [ "10.3.24.10" ] }
  ]
  ```

- This file isn't designed to be hand-edited -- the devices/idlFiles/links arrays reference each other by id, so if you edit it directly and an id gets out of sync, a link can end up pointing to the wrong device. It's safer to change the configuration on screen and save again.

  ```
  "devices": [ { "id": 1, "name": "WCU", ... } ],
  "links":   [ { "srcDevId": 1, "dstDevId": 2, ... } ]
  ```

- Deleting a device, link, IDL, or message that responseRelationships points at later does not automatically drop the relationship from the file — on reload it stays in the list marked "missing reference", and it disappears only when you re-link it to a new message or delete it yourself.

## Writing Rules for IDL Files

This file defines the order and type of fields the payload bytes are split into, and which message ID corresponds to which struct -- without an IDL, step ④'s timeline always shows the payload as raw bytes only, and no periodic/continuity graphs are ever produced. It resembles real C struct syntax, but rather than compiling the whole thing, it's a dedicated parser that only recognizes struct declarations and a handful of markers (tags) attached in // comments.

**How to use**

1. [Required] There must be at least one struct declaration, and exactly one of them must be the "header struct" -- marked with a // $()$ comment to the right of its declaration line. Without this tag anywhere, the whole file fails to parse.

   ```
   struct WCU_CMS_HEADER        // $()$   ← the tag that marks this as the header struct
   {
       unsigned char MsgId;     // $()$
       unsigned char CrcError : 1;
       unsigned char Reserved : 7;
   };
   ```

2. [Required] One field inside that header struct must also carry a // $()$ comment -- this is the field actually read at runtime to determine "which message number is this payload," and without it, parsing likewise fails.

   ```
   struct WCU_CMS_HEADER        // $()$
   {
       unsigned char MsgId;     // $()$   ← the tag that marks the field the message ID is read from
       unsigned char CrcError : 1;
       unsigned char Reserved : 7;
   };
   ```

3. [Required] A field's type must be one of the fixed char/short/int/long/float/double-family names, or the name of another struct already declared in this file -- using an undefined type, e.g. through a typo, is treated as an error.

   ```
   struct WCU_CMS_ENGAGE           // $(1)$
   {
       WCU_CMS_HEADER      header;       // an already-declared struct name
       int                 TargetId;     // a fixed primitive type name
       double              TargetRange;  // a fixed primitive type name
   };
   ```

4. [Required] When structs reference each other as fields, there must be no circular reference back to itself; a bitfield can't be declared on a struct type; a bitfield's width can't exceed its underlying type's full bit size; and a single field can't be declared as both an array and a bitfield at once -- all of these are treated as errors, making that IDL unusable.

   ```
   struct A { B b; };            // A contains B
   struct B { A a; };            // B contains A again → circular reference error!

   struct C {
       unsigned char flag : 1;   // OK: bitfield on a primitive type
       B              bad  : 2;  // error: a struct type cannot be a bitfield
   };
   ```


**Notes**

- //+PACK_SIZE=N defaults to 1 (no padding) if omitted, but declaring it as anything other than 1 makes the load itself fail with an error -- this program only ever decodes with no padding (1-byte alignment), so an IDL whose real protocol does have padding is safer rejected outright than silently misdecoding every field after the first padding gap. //+MOST_BYTE=true|false defaults to true (big-endian) if omitted.

  ```
  //+PACK_SIZE=1
  //+MOST_BYTE=true
  //PERIODIC=[2,3]
  ```

- [Optional] The //PERIODIC=[N,N,...] declaration specifies which message IDs are periodic -- without it, every message is treated as non-periodic and excluded from step ④'s periodic/continuity graphs.

  ```
  //PERIODIC=[2,3]
  // message IDs 2 (WCU_CMS_STATUS_REPORT) and 3 (WCU_CMS_ROUND_INFO) are periodic messages
  ```

- [Optional] A message struct is tagged to message ID N by attaching a // $(N)$ comment to its struct declaration line -- even with none of these tags at all, the file itself is still read, but no message will actually be decoded with that IDL. This message struct's first field should start with the header struct's type -- since the decoder always reads the header starting from the very beginning of the payload, if the first field isn't the header, every field after it ends up at the wrong offset and is misinterpreted.

  ```
  struct WCU_CMS_STATUS_REPORT    // $(2)$   ← registered as message ID 2
  {
      WCU_CMS_HEADER      header;   // the first field is the header struct type
      st_WCU_CMS_STATUS   status;
  };
  ```

- [Optional] Attaching msb=value, bitsize=bit-count (or lsb=value) and unit="unit" in a field's trailing comment converts the raw integer value into a real physical quantity and shows it with its unit in step ④'s detail screen and continuity graphs -- without it, the raw value is simply shown as-is, with no effect on parsing itself.

  ```
  struct WCU_CMS_ROUND_INFO       // $(3)$
  {
      WCU_CMS_HEADER header;
      unsigned int   RoundId;
      int            Altitude_raw;  // lsb=0.1, unit="m"
      int            Velocity_raw;  // lsb=0.1, unit="m/s"
  };
  // Altitude_raw=12345 → the detail view shows it converted as "1234.5 m".
  ```

- Declaring a struct with the same name twice makes the later declaration overwrite the earlier one, and tagging the same message ID on two different structs also makes the later one win -- both cases are shown only as warnings, not errors, so they're easy to miss.

  ```
  // declared first:
  struct WCU_CMS_ENGAGE // $(1)$
  { WCU_CMS_HEADER header; int TargetId; };

  // same name redeclared → overrides the earlier definition:
  struct WCU_CMS_ENGAGE // $(1)$
  { WCU_CMS_HEADER header; double NewField; };
  ```

- [Optional] Instead of "struct NAME {...};", you can also use C's typedef struct syntax — anonymous or tagged, opening brace on the same line or the next line, either works. The tag name (MESSAGE_TAG in the example below), however, cannot be used as another field's type — the name actually used is always the alias (MESSAGE) after the closing brace.

  ```
  typedef struct
  {
      unsigned char value;
  } MESSAGE;              // the name is decided here (the alias)

  typedef struct MESSAGE_TAG {   // the tag name is informational; it cannot be used as a type
      unsigned char value;
  } MESSAGE2;
  ```

- [Optional] In a typedef struct, the "// $()$" (header tag) and "// $(N)$" (message-ID tag) can go on either the opening line ("typedef struct { // $(N)$") or the closing line where the name is decided ("} NAME; // $(N)$"). If the two lines carry different tags, the closing line's tag is used and a warning is shown.

  ```
  typedef struct { // $()$         ← the tag may go on the opening line
      unsigned char MsgId; // $()$
  } HEADER;

  typedef struct
  {
      HEADER header;
      unsigned char value;
  } MESSAGE_A; // $(1)$           ← or on the closing line
  ```

- [Optional] You can also write an entire typedef struct on one line — but because C++ treats everything after "//" as a comment for the rest of the line, a single-line declaration can't carry per-field comments ($SWITCH$/$CASE(n)$/msb=.../"// $()$", etc.) — use the multi-line form if you need per-field comments. A bodyless "typedef struct TAG ALIAS;" form (an alias only, pointing at an already-defined struct) is also supported — here TAG must be a name defined earlier as "struct TAG { ... };", not a typedef struct's own tag name.

  ```
  struct ITEM { unsigned char a; unsigned short b; };

  // whole declaration on one line (no per-field comments; a tag on the struct itself is fine)
  typedef struct { unsigned char a; unsigned char b; } SIMPLE; // $(9)$

  // pure alias -- a new name that copies ITEM's fields as-is
  typedef struct ITEM ITEM_ALIAS;
  ```

- [Optional] Adding "// $SWITCH$" after a field lets you build a conditional structure where, based on that field's actual value, only one of the "// $CASE(n)$"-tagged fields in the same struct is active -- a CASE field doesn't need to sit right next to the SWITCH field, and ordinary fields may appear between them. A struct can, however, have at most one $SWITCH$. The SWITCH field must be an integer type (or integer bitfield); arrays, structs, and floating-point types can't carry this tag. CASE values use the same syntax as message IDs (decimal, or hex with a 0x prefix), plus a leading '-' for negative values -- but if the SWITCH field is a bitfield, it's never sign-extended, so a negative CASE can never actually match (loading still succeeds, just with a warning) -- use a plain signed field instead.

  ```
  struct MESSAGE // $(1)$
  {
      HEADER  header;
      uint8_t mode;       // $SWITCH$
      TYPE_A  dataA;      // $CASE(1)$
      TYPE_B  dataB;      // $CASE(2)$
      uint16_t tail;
  };
  // with mode==1 only dataA appears in the detail view/graphs; with mode==2 only dataB

  // CASE fields may be scattered -- they only have to be in the same struct
  struct MESSAGE2 // $(2)$
  {
      HEADER  header;
      uint8_t mode;       // $SWITCH$
      uint8_t common1;
      TYPE_A  dataA;      // $CASE(1)$
      uint8_t common2;
      TYPE_B  dataB;      // $CASE(2)$
  };
  ```

- [Optional] A $CASE(n)$ field isn't a C union -- it's a "conditional field". Every field (SWITCH, CASE, or ordinary) decodes in declaration order at its own position, and only a CASE field that doesn't match the SWITCH value is skipped entirely, consuming zero bytes. So the actual byte position of a field that follows a CASE field (tail, in the example above) depends on which CASE was active in that occurrence -- if dataA (1 byte) is active, tail starts right after dataA; if dataB (2 bytes) is active, tail starts right after dataB. Decoding as a whole doesn't fail even when no CASE matches the actual SWITCH value (those fields are simply treated as absent, and the fields after them continue decoding normally). A CASE that isn't active never appears in the detail view, graphs, or field-value filters -- it's treated as if it doesn't exist.
- [Optional] $SWITCH$/$CASE(n)$ is separate from "additional message identifiers" (the feature that further splits the same message ID by a field's value) -- an identifier only splits an already-fixed structure by value, while SWITCH/CASE declares in the IDL that the payload's byte structure itself changes depending on a value. You can use both features on the same message.

---

[← Overall Workflow](01_workflow.md) · [Contents](../../README.en.md#user-guide) · [① Devices / IDL →](03_step1.md)
