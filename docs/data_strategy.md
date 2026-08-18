# Data Strategy

## Purpose

The goal of this document is to define the data requirements needed to build Athlete Career Intelligence.

Before developing any model, dashboard, or AI feature, it is necessary to establish a clear relationship between:

```text
Player Information
      ↓
Career Outcomes
      ↓
Future Value Score
```

The data strategy focuses on collecting reliable and consistent datasets that enable the analysis of player development from college basketball to the NBA.

---

# Guiding Principle

The project does not aim to predict basketball statistics.

The project aims to identify players capable of generating high future professional value.

Therefore, every dataset included in the platform must contribute to answering one of the following questions:

1. Who is this player?
2. How did this player perform?
3. What happened next in his career?
4. How valuable was that career?
5. What factors contributed to that outcome?

---

# Initial Scope

## Version 1

The first version of Athlete Career Intelligence will focus exclusively on:

```text
NCAA → NBA
```

This scope has been intentionally limited to reduce complexity and validate the project's core assumptions.

Future versions may expand to:

- G League
- EuroLeague
- International Leagues
- NBA Free Agency
- Contract Analytics
- Team Intelligence

These areas are outside the scope of the MVP.

---

# Core Data Model

The platform requires four primary datasets.

## Dataset 1: NCAA Players

Purpose:

Understand player profiles before entering professional basketball.

Key Variables:

### Identification

- Player Name
- Player ID
- Season
- Team
- Conference

### Demographics

- Age
- Height
- Weight
- Position

### Performance

- Games Played
- Minutes Played
- Points
- Rebounds
- Assists
- Steals
- Blocks
- Turnovers

### Shooting

- FG%
- 3PT%
- FT%

### Advanced Metrics

- Usage Rate
- Offensive Rating
- Defensive Rating
- Win Shares (if available)
- BPM (if available)

---

## Dataset 2: NBA Draft

Purpose:

Measure market perception at the moment a player enters professional basketball.

Key Variables:

- Draft Year
- Draft Position
- Draft Round
- Draft Team

This dataset will help identify:

- Undervalued players
- Overvalued players
- Draft steals
- Draft busts

---

## Dataset 3: NBA Career Outcomes

Purpose:

Measure the real value generated after entering the NBA.

Key Variables:

### Career Longevity

- Seasons Played
- Games Played
- Minutes Played

### Production

- Career Points
- Career Rebounds
- Career Assists

### Advanced Metrics

- Career Win Shares
- Career VORP
- BPM

### Recognition

- All-Star Appearances
- All-NBA Teams
- Awards

---

## Dataset 4: Future Value Target

Purpose:

Create the target variable used by predictive models.

Output:

```text
Future Value Score
```

The exact formula will be defined in:

```text
future_value_definition.md
```

---

# Entity Structure

The entire platform revolves around a single entity:

## Player

Every dataset must connect through a unique player identifier.

Conceptually:

```text
Player

├── NCAA Career
├── Draft Information
├── NBA Career
└── Future Value Score
```

This structure will allow all platform modules to work from a common data foundation.

---

# Data Architecture

## Raw Layer

Contains original downloaded datasets.

Location:

```text
data/raw/
```

Characteristics:

- Unmodified
- Source of truth
- Version controlled

---

## Clean Layer

Contains cleaned and standardized data.

Location:

```text
data/processed/
```

Characteristics:

- Standardized player names
- Corrected data types
- Missing values treated

---

## Analytics Layer

Contains modelling-ready datasets.

Location:

```text
data/features/
```

Characteristics:

- Engineered features
- Similarity variables
- Model inputs

---

# Initial Research Questions

Before building models, the following exploratory questions should be answered.

## NCAA to NBA Pipeline

- How many NCAA players reach the NBA?
- How many players remain in the NBA for at least three seasons?
- Which college statistics correlate most strongly with NBA success?

---

## Draft Analysis

- How strongly does draft position predict future value?
- Which draft positions historically generate the best return?
- What are the most common characteristics of draft steals?

---

## Career Outcomes

- What do successful NBA careers look like?
- How should career success be measured?
- Which metrics best capture long-term value?

---

# Data Sources (Candidate Sources)

The following sources will be evaluated during the acquisition phase.

## NCAA

- Sports Reference
- College Basketball Reference
- Kaggle NCAA datasets

## Draft

- Basketball Reference Draft Database

## NBA

- Basketball Reference
- NBA API
- Kaggle NBA Databases

The final source selection will prioritize:

- Data quality
- Historical coverage
- Ease of maintenance
- Reproducibility

---

# MVP Success Criteria

The data strategy will be considered successful when the project is capable of creating a single dataset where every player contains:

```text
NCAA Profile
      +
Draft Information
      +
NBA Career Outcome
      +
Future Value Score
```

Once this dataset exists, all major components of Athlete Career Intelligence can be developed:

- Market Dashboard
- Player Cards
- Similarity Engine
- Career Simulator
- AI Scout Reports
