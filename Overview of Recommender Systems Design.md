# Preface

This documents lists some of the popular techniques for building a recommender system for a real-world scenario.

The motivational scenario provided is as follows:

We have job seekers registering on a popular job search portal. These users submit a substantial amount of detail about their education and work experience. The portal has about a million registered users. 

Build a recommender system that does 2 things for each user:

1) Curates a list of best matching jobs.  
2) Identifies gaps between the user’s profile and the job, and prescribes an action plan to close the gaps.

Although the design overviews presented in this document are tuned for the above scenario, it can be applied to build recommender systems for any other scenario in any domain (with few changes).

# Data Overview

The following table lists some of the common data available for a job portal:

| Category | Sub-Category | Examples |
| :---: | ----- | ----- |
| **User Data** | Demographics | Age, Gender, Current and Preferred Locations |
|  | Education | Degree, Institution, Year, Certifications |
|  | Experience | Job Titles, Companies, Tenure, Achievements |
|  | Skills | Explicit skills, Proficiency Levels, Languages |
|  | Resume | Free-text Document |
|  | Preferences | Desired Roles, Industries, Salary, Work Type |
| **Job Data** | Job Title | "Software Engineer", "Backend Developer" |
|  | Requirements | Degree, Certifications, Experience Level |
|  | Skills | "Python, SQL, Leadership" |
|  | Job Description | Responsibilities, Objectives |
|  | Company Info | Name, Industry, Size |
|  | Metadata | Salary, Location, Remote Option, Post Date |
| **Interaction Data** | Implicit Feedback | Application History: Applied, Viewed, Saved, Rejected, Interviewed |
|  | Optional Explicit Feedback | Star Rating, Thumbs Up / Down, Relevance Labels |

## 

# Solution Overview

The solution for a recommender system (RS) involves many components, which come under the following 3 pipelines / stages:

- RS Data Pipeline  
- RS Model Pipeline  
- RS Consumption Pipeline

This document outlines the solution design overview for the first two stages, the consumption stage is not outlined here. Consumption is usually in the form of email to the customer and the UI of the job portal (dashboard).

## 1\. Data Pipeline

### 1.1. Data Transformation Steps

Some of the common data cleaning and transformation steps are outlined here:

| Step | Description | Techniques |
| :---: | ----- | ----- |
| **Deduplication** | Remove duplicate user and job records | Drop records, Database constraints |
| **Standardization** | Normalize inconsistent attribute formats and values | Rule-based mapping, Reference lists |
| **Text Cleaning** | Prepare free-text for analysis; remove noise | Lowercasing, stopword removal, spellcheck |
| **Skill Normalization** | Map synonyms and variants to canonical skills | Ontologies, dictionaries |
| **Date Parsing** | Uniform date formats, validate date entries | Date libraries |
| **Imputation** | Address blanks or incomplete records | Filling, flagging, removal |
| **Encoding** | Convert categorical variables to numerical formats | One-hot, label encoding |
| **Vectorization** | Convert user and job profile data (after extraction and aggregation) into vectors for **content-based filtering** | TF-IDF, Word2Vec, Transformer-based embeddings |
| **Interaction Matrix** | Construct user-job interaction matrix for **collaborative filtering** | Matrix construction, sparse formats |

### 

### 1.2. Transformed Data Examples

#### 1.2.1. Profile data for vectorization

`User Profile:`   
`[NumPy, Python, SQL] | Data Analyst | 2 yrs | M.Sc. | Finance | Boston` 

`Job Profile:`   
`[Python, SQL, Tableau] | Data Analyst | 3 yrs | B.Sc. | Finance | Boston`

The techniques used for extraction (used for building an aggregated profile as shown above) are: 

- Structured extraction: Ruled-Based (regex, keywords, mappings) and Template-Based  
- Conventional NLP extraction: PoS tagging, Dependency Parsing, NER  
- LLM-powered NLP extraction: Using the power of LLMs to extract key data pieces

#### 1.2.2. User-job interaction matrix

|  | `Job1` | `Job2` | `Job3` | `…` |
| :---- | :---- | :---- | :---- | :---- |
| `UserA` | `5` | `2` | `0` |  |
| `UserB` | `0` | `1` | `-1` |  |
| `UserC` | `0` | `0` | `0` |  |
| `…` |  |  |  |  |

## 

Here, the numbers in the cells represent a certain type of interaction. For example, a value of 5 may mean that the user applied to the job, and a value of 0 may mean no interaction. 

These interaction codes (values) need to be planned and created for building such a matrix.

## 2\. Model Pipeline

### 2.1. Components Overviews

#### 2.1.1. Hybrid Recommender Component {#2.1.1.-hybrid-recommender-component}

For designs that use a weighted hybrid model of collaborative and content-based filtering, the following 3-step process is applicable:

1. Content-Based Filtering

   After extraction and aggregation, the profile data can be vectorized and similarity can be computed (using cosine similarity or neural-network model) to implement content-based filtering.

2. Collaborative Filtering

   After constructing the Interaction Matrix, algorithms such as Alternating Least Squares (ALS, which belongs to the Implicit Matrix Factorization family) or Nearest Neighbors can be used to implement collaborative filtering.

3. Hybridization

   First, design a weighted aggregation strategy, such as assigning larger content-based weights for new users and assigning larger collaborative weights for engaged users. 

   Then, apply the formula : 

   `Score(u, j) = w_content       × Score_content(u, j)` 
               `+ w_collaborative × Score_collaborative(u, j)` 

   Here, Score(u, j) would be the final hybrid recommendation score for user (u) and job (j).

#### 2.1.2. LLM Component

Generally, good prompt engineering is usually sufficient to steer an LLM towards the desired behaviour. In some cases, however, LLM fine-tuning may be more effective and efficient.

#### 2.1.3. Feedback Component

For non-agentic systems, a simple feedback loop can be implemented using rule-based systems.

For agentic systems, RLHF feedback integration can be implemented using algorithms like Contextual Bandits and Q-Learning.

#### 

### 2.2. Design Overviews

Provided below are 3 design overviews for building the model pipeline.

#### 2.2.1. Design \#1

In this design, the RS is built in 2 parts (non-agentic):

- Part 1 (Job Recommender) of the RS is built using a conventional, tried-and-tested, industry-standard technique: a weighted hybrid model of collaborative and content-based filtering. An overview for implementation is provided in [Section 2.1.1. \- Hybrid Recommender Component](#2.1.1.-hybrid-recommender-component). The output would be a list of (user, job) pairs.  
    
- Part 2 (Gap Identification & Action Plan) is built using an LLM. The output would be a list of identified gaps and action plan with a summary, for each (user, job) pair. 

![Design #1 - Non-Agentic](https://drive.google.com/uc?export=view&id=1AKHNZU47HiQ8ebSZmGX_lLrWblM2YyI_)

#### 2.2.2. Design \#2

In this design, the RS is built using a fairly standard agentic approach (with RLHF). We have 2 sets of agents:

- Agents (one agent per user, adapting over time) to curate the list using a weighted hybrid model of collaborative and content-based filtering. RLHF is used here to train the agent’s policy for job ranking strategy: both historical data and user feedback are used to generate reward / penalty signals and update weights and selection policies.  
    
- Agents (individual or shared agents are equivalent here in terms of results, as this would be a stateless operation) to identify the gap and prescribe an action plan: using an LLM. RLHF is used here to adjust prompt templates, which another LLM agent can take charge of.

![Design #2 - Agentic](https://drive.google.com/uc?export=view&id=1qagvEVnf8341mxJ4yJdn4mwSCGUU4Zi9) 

#### 2.2.3. Design \#3

In this design, the RS is built using a more bleeding-edge agentic approach (with RLHF). We have a team of LLM agents that curate the jobs, identify the gaps and also prescribe an action plan. 

Each agent has a specialized responsibility and interacts in a multi-agent dialog, enabling stepwise refinement. They are also trained and improved via RLHF.

![Design #3 - Multi-Agent](https://drive.google.com/uc?export=view&id=1o37GX0EHJqFUlYegl97bwCzYTKlXRYhi)

### 2.3. Designs Comparison

We can find a summary of the pros and cons of the designs in the following table:

| Design | Pros | Cons | Ideal Use Cases |
| :---- | :---- | :---- | :---- |
| **Design \#1: Non-Agentic** | \- Mature, reliable, explainable \- Low complexity, proven scale \- Stable operation \- Lower cost | \- Less adaptive \- Feedback loop is slow  \- Less personalised | \- Mainstream job boards \- Services focused on scale and efficiency \- When explainability and simplicity are priorities |
| **Design \#2: 2 Sets of Agents with RLHF** | \- One agent per user enables tailored curation that improves over time \- RLHF enables continuous personalization of jobs curation \- LLM action planner's clarity and relevance improve rapidly via RLHF and prompt tuning | \- Infrastructure and state management per user \- Division between curation (ML-based agent) and planning (LLM) may make QA more difficult \- Prompt template management adds a layer of complexity | \- Personalized career coaching job platforms \- Environments with diverse job/skill requirements \- Systems able to invest in per-user modeling and feedback loops |
| **Design \#3: Multi-Agent LLM Ensemble with RLHF** | \- Maximum bias correction and self-improvement \- Best for nuanced and high-stakes contexts \- RLHF on every reasoning step for strong human alignment \- Can force high transparency and explainability \- Particularly strong for mixed goal (curation, diagnostics, advice, QA, coaching) | \- Higher cost \- Coordination across agents and managing their dialogs is complex \- Requires large, high-quality feedback data for RLHF \- Debugging can be challenging | \- Mission-critical or premium expert coaching platforms \- Where quality and oversight are paramount \- Environments demanding both best-in-class curation and prescriptive guidance |
