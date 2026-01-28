# Estimating socio economical paramaters for residential buildings


### Introduction

The main idea of estimating occupancy for buildings lies in disaggregation of the national statistics into buildings proportionally to a selected mix of building attributes, e.g., the gross floor area of buildings. The base idea of dispersing/disaggregating census data to estimate building occupancy or population density in smaller grids is not new, and is used in several papers, including in [a GIS Approach to Estimation of Building Population for Micro-spatial Analysis](https://www.cdema.org/virtuallibrary/images/A%20GIS%20Approach%20to%20Estimation%20of%20Building.pdf), [Estimating Population a 100-meter-geographical grid](https://www.unica360.com/estimating-population-at-a-100-meter-geographical-grid), and [Disaggregating population data for assessing progress of SDGs: methods and applications](https://www.tandfonline.com/doi/epdf/10.1080/17538947.2021.2013553?needAccess=true).

When disaggregating census/statistical data into buildings it is important to measure how many inhabitants would a building attract. In the method below the gross floor area (GFA) of residential buildings is used to provide this metric of attraction. In simple terms comparing two buildings A and B, with A being twice as big as B we should expect around twice as many people living in building A than in building B. The gross floor area of buildings is defined as the sum of floor area of buildings extending to the outer face of the external walls for each floor. In the case of the OBI tool, it can be computed as the footprint area (a*b) times the estimated number of floors (f):

![pop_break_down](/images/a.png)

Additionally, a more complex approach is defined, where informal settlements are taken into consideration, where it is assumed, that a building would attract more inhabitants inside informal settlements, than a building from a formal neighborhood with the same gross floor area.

In the rest of the text the following is silently assumed:
- People are distributed into buildings uniformly based on their size.
- People live in residential buildings only.

### Splitting the Selected Region Into Areas

Generally, population information is collected in census and provided as aggregated results for certain administrative areas. For this exercise it is important to break down any region for the lowest possible available administrative level, especially taking into account of urban and rural boundaries, as buildings can behave differently in those settings.

In our case the local government has provided ward level boundaries and aggregates for the cities. Wards are very small administrative boundaries, therefore this aggreagted data is more precise than city-level aggregates. In our case these boundaries also adhere to city boundaries, hence reliable depicting rural-urban boundaries as well.

### Disaggregating Population For a Given Area

Let $Bld$ denote the set of all residential buildings inside a given area, $GFA_B$ denote the gross floor area for any building $B$ and $Pop$ denote the total population of the area. Then, for every building $B$ the number of inhabitants can be computed using the following formula:

$$ ROcc_B = \frac{Pop}{\sum_{B' \in Bld}{GFA_{B'}}} GFA_B. $$

Additionally, if the set of all buildings is split into formal ($Frm$) and informal ($Inf$), such that $Frm \cup Inf = Bld$ and $Frm \cap Inf = \emptyset$, assuming $K$ times more people live in informal areas for the same square meter compared to formal areas the formula can be further adjusted:

$$ ROcc_B = \frac{Pop}{K \times \sum_{B' \in Inf}{GFA_{B'}} + \sum_{B' \in Frm}{GFA_{B'}}} (IS_B \times K) GFA_B, $$

where $IS_B$ represents the informal classification of the building, $0$ for formal buildings and $1$ for informal ones.

The factor describing how much more densily are informal settlements populated can be deduced by comparing statistics describing how much square meters inhabitants have in formal and informal settlements. This number can change from country to country, currently it is set to be $K=3$.

These formulae return real numbers. If the number would be rounded to integers the are could be overestimated significantly. As an example imagine an area with 10  buildings of the same size, where 15 people dwell. Using the methodology above each building would have an estimated occupancy of 1.5, which rounded would yield 2 people. So in our small example the area would be estimated to have 20 population, 5 more than the real number.


### Avoid Rounding

The main idea presented in this section to estimate population of buildings as an integer number is the under-estimation of building population, hence under-estimating the entire area, leading certain amount of people "unassigned" to compensate with in case buildings "deserve" additional occupants. This idea is not new, it is used, for example, in parliamentary voting schemes in countries with proportional voting systems to distribute parliamentary seats between parties, in which case non-real numbers need to be "rounded" to integer numbers while also filling exactly the given number of parliamentary seats.

There are different approaches, how to asociate integer numbers based on real ones into a set of entities (this case buildings) to retain a selected sum. The approach presented below is a simple naive disaggregation approach.

As a first step the real occupancy of buildings is split into its integer floor ($IOcc_B = \lfloor ROcc_B \rfloor$) and the remaining floating part ($FOcc_B = ROcc_B - \lfloor ROcc_B \rfloor < 1$). Summing up the under-estimated integer occupancy for all buildings inside area the following holds:

$$Pop - |Bld| < \sum_{B \in Bld} IOcc_B \leq Pop.$$

Leading to a smaller amount of "unassigned" people than the number of buildings, enabling to compensate some buildings by at most one resident.

Last, it needs to be defined, which buildings "deserve" additional compensation. Assuming, that an additional inhabitant is more impactful when the population of the building is low (an additional population is more impactful in terms of energy consumption in a house of 5 than in a multi-apartment building housing 200) the factor, how much buildings deserve an additional compensation can be computed as a factor $\frac{FOcc_B}{IOcc_B}$ describing how much additional value was lost during the under-estimation process per capita.

Additionally, buildings with zero $IOcc_B$ are compensated by default.

Overall, the first $Pop-∑_{B \in Bld} IOcc_B$ are compensated. It is easy to see, that the sum of all estimated occupants in the area sums up to the exact population of the area. Formula to use:

$$\begin{cases}\lceil ROcc_B \rceil &\quad\text{B "deserves" compensation}\\
       \lfloor ROcc_B \rfloor &\quad\text{otherwise.} \\ \end{cases} $$


The image illustrates the population disaggregation model, which uses residential building size (Gross Floor Area) as a direct proxy for capacity. Based on the assumption that larger buildings house more people, the model distributes the total population proportionally:

![pop_break_down](/images/Pop.png)


### Future Considerations

Adding an additional stepinto the disaggregation process, where first the number of households (flats) inside a building would be estimated and later this number would be used (instead of GFA) to disaggregate population. This adds an additional complexity to the computation, but it would solve the problem, where buildings having - as an example - 60 and 80 square meters could have significant difference in occupancy, while in many cases both ould be considered a single household buildings. But the coplexity to assign households to buildings can be showcased on the example above as well. Can we be sure, that an 80 square meter building contains only 1 household and not 2?