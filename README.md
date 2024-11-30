# obsidian-stock-portfolio v1.0.0
Script for to display stock holdings with Dataview and Chart plugins

![demo](https://github.com/user-attachments/assets/dd98f340-2403-42f7-90c7-04ee4fcf63e9)

## Instructions
You will want a directory, for example named "Orders", that will have 1 note per ticker. You will need to update the script to point to the correct `tickerPath`, which is the first line in the options in `Stocks Summary.md`.
Each of these notes will need the following properties:

![image](https://github.com/user-attachments/assets/e0bb3c6e-7040-4544-b89b-97d0479f69d4)

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
