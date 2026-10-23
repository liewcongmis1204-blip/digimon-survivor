# Digimon Survivor — 进化配置说明

本说明对应 `assets/digimon-evolution-config.js` 和当前游戏脚本。当前版本先接好规则框架，不新增关卡，也不改动相机／大世界、升级三选一、既有进化光环效果和本地最高纪录存档。

## 目前已接入

- 3 个版本，各 32 只数码兽，共 96 只：自然之魂、深海救星、恶梦士兵。
- 原始的 `id / name / stage / attr / spriteFile / from / branches` 数据保留。
- 每只数码兽运行时拥有独立的 6 个技能槽、1 个合成必杀配方模板，以及每条出边独立的分支条件。
- 能量满值为 100；敌人击杀和经验球都会增加进化能量。
- 已进化形态的能量下降到 15 以下时，会依照当前数码兽 `from` 回退一级，并同步 sprite 名称、属性和基础属性比例。
- 疫苗／病毒／数据属性克制：Va 克 Vi，Vi 克 Da，Da 克 Va。
- 普通敌人和 Boss 从全部 96 只中按阶段权重抽取；生存时间越长，完全体与究极体权重越高。
- 控制台进化日志格式：`xxx 进化 → xxx，分支条件匹配成功`。

## 填技能名称与必杀配方

打开 `assets/digimon-evolution-config.js`，在文件末尾找到：

```js
window.DIGIMON_SKILL_OVERRIDES = window.DIGIMON_SKILL_OVERRIDES || {};
```

把这一行改成一个对象，并在里面只写你要编辑的数码兽即可。例如：

```js
window.DIGIMON_SKILL_OVERRIDES = {
  "tanemon": {
    "skillPool": {
      "1": {
        "id": "tanemon_leaf_cut",
        "name": "叶刃",
        "description": "植物属性攻击",
        "tags": ["plant"],
        "enabled": true
      },
      "2": {
        "id": "tanemon_root_bind",
        "name": "根缚",
        "description": "控制目标",
        "tags": ["control"],
        "enabled": true
      }
    },
    "ultimateRecipe": {
      "id": "tanemon_ultimate_combo",
      "name": "森林终极连携",
      "requiredSkillIds": ["tanemon_leaf_cut", "tanemon_root_bind"],
      "enabled": true,
      "minStage": "rookie",
      "sealOnDevolution": true
    },
    "branchConditions": {
      "kabuterimon": {
        "requiredSkillIds": ["tanemon_leaf_cut"],
        "priority": 10
      },
      "togemon": {
        "requiredSkillIds": ["tanemon_root_bind"],
        "priority": 10
      }
    }
  }
};
```

注意事项：

- `skillPool` 的键 `"1"` 至 `"6"` 对应六个技能槽。
- `requiredSkillIds` 必须与技能槽的 `id` 一致。
- `requiredUltimateRecipeIds` 用来要求已合成的必杀配方；`anySkillIds` 用来匹配任意一个技能；`excludedSkillIds` 用来排除技能。
- `priority` 越高，在多个分支条件都匹配时越优先。
- 若某只数码兽没有填任何分支条件，系统会从它的有效 `branches` 中随机选一个，并仍会打印分支判定日志。
- 以下控制台 API 可用于测试和编辑当前页面运行中的状态，但刷新页面后，调试控制台做的修改不会自动写回 JS 配置文件：
  - `DigimonEvolutionAPI.getState()`
  - `DigimonEvolutionAPI.setSkillSlot("tanemon", 1, {id:"tanemon_leaf_cut", name:"叶刃", enabled:true})`
  - `DigimonEvolutionAPI.setBranchCondition("tanemon", "kabuterimon", {requiredSkillIds:["tanemon_leaf_cut"], priority:10})`
  - `DigimonEvolutionAPI.setUltimateRecipe("tanemon", {id:"tanemon_ultimate_combo", name:"森林终极连携", requiredSkillIds:["tanemon_leaf_cut"], enabled:true, minStage:"rookie", sealOnDevolution:true})`
  - `DigimonEvolutionAPI.setEvolutionEnergy(100)`

## 添加角色 PNG 贴图

- 原始选角的亚古兽与加布兽目前使用 `assets/agumon.png` 与 `assets/gabumon.png`。
- 配置中其余角色的图片按 `assets/digimon/<spriteFile>` 加载。例如 `greymon.png` 应放在 `assets/digimon/greymon.png`。
- `currentSprite` 会在进化／退化时切换成配置指定的 `spriteFile`。如果目标 PNG 尚未上传，游戏会暂时回退显示原始搭档图；Boss／普通敌人也会先使用现有的简化图形。

## 原始配置里未定义的分支目标

你提供的 96 只清单中，有 22 个 `branches` ID 没有对应的 `digimonList` 记录。为了不擅自增删你的 96 只原始数据，游戏会在控制台列出这些目标，并在运行时跳过它们：

`palmonperfect`, `skullgreymon`, `warumonzaemon`, `tokomon`, `granukuwagamon`, `gigasmon`, `monzeemon`, `pororotamon`, `metalogreimon`, `banchomamemon`, `syakkomon`, `marinangemon`, `sukamon`, `jijimon`, `mojamon`, `weregarurumon`, `marindevimon`, `dokugumonperfect`, `metalgreymonvirus`, `blackmetalgarurumon`, `omegamon`, `malomyotismon`.

原始的 `from: ["egg"]` 也被保留；本次 96 只清单没有单独的 egg 记录，所以不会凭空创建蛋阶段物种。
