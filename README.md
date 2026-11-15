#  IPL Dashboard – Power BI Analysis

An interactive **Power BI dashboard** that analyses Indian Premier League (IPL) match and ball-by-ball data to uncover insights on team performance, player statistics, toss impact, venue trends and season patterns.

**Created by:** Abhay Chauhan
**Tools:** Power BI · Power Query · DAX
**Dataset:** IPL Matches & Deliveries


---

##  Project Objectives

- Analyse team performances across seasons
- Identify top batsmen and bowlers
- Study the impact of the toss on match results
- Understand season-wise trends
- Analyse venue and city performance

---

##  Dataset

| Dataset | Description |
|---------|-------------|
| **Matches** | Match-level data: teams, toss winner, toss decision, match winner, player of the match, city, venue, season |
| **Deliveries** | Ball-by-ball data (~150K rows): batting/bowling team, batsman, bowler, runs, extras, dismissals |

Key columns used from the Deliveries dataset include `match_id`, `batting_team`, `bowling_team`, `batsman`, `bowler`, `batsman_runs`, `extra_runs`, `total_runs`, `player_dismissed` and `dismissal_kind`.

---

##  Data Cleaning & Transformation

- Handled null values using **Replace Values** and filtering in Power Query
- Corrected data types
- Created **relationships** between the Matches and Deliveries tables
- Built **DAX measures** for KPIs and calculations

---

##  Dashboard Overview

### Dashboard 1: Player & Team Performance

| Visual | Chart Type | What it shows |
|--------|-----------|---------------|
| Batsman Runs | Bar chart | Top run scorers (SK Raina, V Kohli, RG Sharma, etc.) |
| Total Sixes & Fours | Stacked bar chart | Boundary distribution among leading batsmen |
| Player of the Match | Bar chart | Frequency of Player of the Match awards |
| Winner by Toss | Pie chart | Distribution of toss wins among franchises |

### Dashboard 2: Toss, Venue & Trends

| Visual | Chart Type | What it shows |
|--------|-----------|---------------|
| Toss Decision | Pie chart | Proportion of teams choosing to bat vs field |
| Winner by Toss Winner | Bar chart | How often toss winners also won the match |
| Match City | Map | Geographic distribution of IPL matches |
| Batsman Runs by Total Runs | Line/area chart | Relationship between batsman runs and total runs per ball |

---

##  Key Insights

- **Top run scorers:** SK Raina, V Kohli and RG Sharma lead the run charts, showing consistency across seasons.
- **Boundary hitters:** G Gambhir, SK Raina and DA Warner stand out in fours; CH Gayle is prominent for sixes.
- **Match impact:** CH Gayle, AB de Villiers and MS Dhoni are among the most frequent Player of the Match winners.
- **Toss wins:** Mumbai Indians and Kolkata Knight Riders have won the most tosses.
- **Toss decision:** Around 57% of toss winners chose to field and about 43% chose to bat.
- **Geography:** Matches are concentrated in Indian cities, with some played in South Africa and the UAE.

---

##  Business Recommendations

1. **Use venue insights** – Identify stadiums that give teams better winning chances to plan match strategy and focus fan events and promotions.
2. **Focus on star players** – Retain top performers, plan training and attract sponsors.
3. **Engage fans with data** – Share toss, sixes and Player of the Match stats on social media.
4. **Smart recruitment** – Use performance numbers to spot rising talent and make better auction decisions.

---

##  Repository Structure

```
├── IPL_Dashboard.pbix            # Power BI dashboard file
├── IPL_Dashboard_Document.pdf    # Dashboard documentation with screenshots
├── IPL_Analysis.docx             # Detailed analysis report
├── data/
│   ├── matches.xlsx              # Matches dataset
│   └── deliveries.xlsx           # Deliveries dataset
└── README.md
```

---

##  How to Use

1. Clone or download this repository:
   ```bash
   git clone https://github.com/<your-username>/<your-repo-name>.git
   ```
2. Install [Power BI Desktop](https://powerbi.microsoft.com/desktop/) (free, Windows).
3. Open `IPL_Dashboard.pbix`.
4. If prompted, update the data source path: **Home → Transform data → Data source settings**, and point to the files in the `data/` folder.
5. Click **Refresh** and explore the dashboards.

---

##  Conclusion

These dashboards go beyond match results and offer practical lessons for teams and businesses. From planning strategies and retaining top players to engaging fans and making smart recruitment decisions, they turn IPL data into clear actions, making cricket analysis a tool for business growth.

---

##  Author

**Abhay Chauhan**
Feel free to connect and share feedback!

 If you found this project useful, consider giving it a star.
