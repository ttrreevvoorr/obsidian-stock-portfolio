# obsidian-stock-portfolio v1.0.0
Script for to display stock holdings with Dataview and Chart plugins

**Example:**

![image](https://github.com/user-attachments/assets/4e5fbf60-3e28-420a-9fde-4c76c8889de1)

## Instructions
You will want a directory, for example named "Orders", that will have 1 note per ticker. You will need to update the script to point to the correct `tickerPath`, which is the first line in the options in `Stocks Summary.md`.
Each of these notes will need the following properties:

```
---
Shares: 5
Cost Basis: 133.873
Profits:
---
```

| Property   | Description                                  |
| ---------- | -------------------------------------------- |
| Shares     | Number of Shares you own                     |
| Cost Basis | The average cost of the shares you own       |
| Profits    | Any realized gains you've made on the ticker |

### Pugin Dependencies
Plugins you will need to install:

**Obsidian Dataview**
https://github.com/blacksmithgu/obsidian-dataview

**Obsidian-Charts**
https://github.com/phibr0/obsidian-charts
