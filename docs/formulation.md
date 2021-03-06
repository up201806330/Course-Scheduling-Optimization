# Optimization Project
**COURSE SCHEDULING IN THE COMPUTER SCIENCE AND OPTIMISATION DEPARTMENT OF MONFORT COLLEGE**

- Diogo Rodrigues (up201806429)
- Gonçalo Pascoal (up201806332)
- Rafael Ribeiro (up201806330)

---

## Problem Statement Analysis

> Gerardo said, “I suggest that we model it in two stages: in stage 1, we assign courses to the faculty; in stage 2, we assign each course-faculty pair to a class time.” Pedro liked the idea. Gerardo added, “An additional advantage is that we can split the scheduling stage into two separate problems: MWF and TuTh.”

We have decided to follow Gerardo's advice and split this problem into two stages. In the first stage, we assign courses to faculties; more specifically, we find $r_{f,c}$, which is the workload of faculty member $f$ in course $c$ (that is, the number of classes of a course that are lectured by that faculty member).

In the second stage, we can solve the scheduling problem, split into Monday-Wednesday-Friday and Tuesday-Thursday cases separately. This is only possible because there is no full-time faculty that teaches both in MWF and TuTh time slots (we assumed that the faculties which do not appear in a given preference table do not teach in the corresponding days).

> They also agreed that the preferences of senior faculty would be prioritized over those of junior faculty (...) (Table 4).

> (...) the objective of the optimization model in each of the two stages, i.e. the “course assignment” stage and the “scheduling” stage, would be maximizing the total faculty preferences

This requirement is implemented in the main part of the objective functions of both stages, by multiplying preferences by seniority.

### Course Assignment Phase
#### Hard Constraints
> Each faculty is assigned to only the courses that he/she is qualified to teach (Table 5). For example, Dr. Barbosa is qualified to teach all courses except CSO6 and CSO8. 
![](https://i.imgur.com/pR2VsRv.png)

Constraint 1. We are assuming that contingent faculty are qualified to teach any course.

> Each full-time faculty member’s actual course load must be equal to his/her required course load per semester according to their contracts (Table 4).
![](https://i.imgur.com/NyYHPqv.png)

Constraint 2.

> The number of classes assigned to the faculty needs to be equal to the number of classes required for each course (Table 6). For example, the CSO department must offer exactly five classes of CSO3 in the Spring 2023 semester over all faculty due to the student demand.
![](https://i.imgur.com/cmR6dSp.png)

Constraint 3.

#### Soft Constraints
> (...) he would like all courses, and particularly CSO6, to be taught by full-time faculty, if possible

Constraint 5.

> Ideally, the number of preparations (different courses that each faculty taught per semester) for each full-time faculty would be no more than two.

Constraint 6.

> Ideally, no more than two classes of the course CSO3 should be assigned to any full-time faculty because of the extra workload emanating from the lab.

Constraint 7.

#### Preferences

![](https://i.imgur.com/kyISg0Q.png)

### Scheduling Phase

> (...) sixty minutes per class on Monday, Wednesday, and Friday (MWF) or (...) 85 minutes per class on Tuesday and Thursday (TuTh) 

Translated as $\Delta$, a parameter used in constraint 11.

#### Hard Constraints

> At most three course classes can be assigned to the same class period based on allotments determined by the Assistant Dean. In other words, no more than three classes of one or multiple CSO courses can be scheduled in the same class period.

Constraint 10.

#### Soft Constraints

> Most of the students and faculty preferred that their class day began later than 9:00 am and ended before 4:00 pm

Constraint 11.

> (...) it would be reasonable to try to avoid having instructors teach more than two consecutive class periods in a day

Constraint 12.

> In fact, he did not have a pleasant experience with back-to-back classes scheduled in the afternoon

Constraint 13.

### Preferences

![](https://i.imgur.com/GCKLEDT.png)

![](https://i.imgur.com/2SubsVx.png)

## Formulation
### Course Assignment Phase
#### Sets and Indices
- Courses ($C$): $\{CS01, CS02, CS03, CS04, CS05, CS06, CS07, CS08\}$
To simplify, each course can be represented by its number, from 1 to 8.
- Faculty members ($F$): $\{$*Barbosa*, *Castro*, *Gerardo*, *Lameiras*, *Machado*, *Pedro*, *Queirós*, *Soeiro*, *contingent*$\}$
To simplify, this set can be represented by the initials of each name (because initials do not repeat: B, C, G, L, M, P, Q, S), with *contingent* represented as $\alpha$

#### Parameters
$S_f$: Seniority of faculty member $f$
$L_f$: Course load for faculty member $f$, as in, number of classes taught in a week. In practice, for the contingents we assigned a value greater than the total number of classes considering all courses (23), which was needed to relate the $r_{f,c}$ and $b_{f, c}$ decision variables.

$N_c$: Required number of classes for course $c$
$M$: Sum of the required number of classes for all courses

$P_{f,c}$: Preference of faculty member $f$ for course $c$
$Q_{f,c}$: Qualification of faculty member $f$ for course $c$

$X$: Weight that a represents how important it is for course 6 to not be lectured by non-faculty (soft constraint 5)
$W_1$: Weight that represents how important it is that no more than 2 courses are assigned to any full-time faculty (soft constraint 6)
$W_2$: Weight that represents how important it is that no more than 2 classes of the course CSO3 are assigned to any full-time faculty (soft constraint 7)

#### Decision Variables
$r_{f,c}$: Number of classes of course $c$ given by faculty member $f$ 
$b_{f,c}$: Assignment of faculty member $f$ to course $c$, boolean variable that is 1 when $r_{f,c} \geq 1$ and 0 when $r_{f,c} = 0$

$u_f$: Number of course preparations above 2, for each full-time faculty member $f$. *Used for soft constraint 6*

$v_f$: Number of classes of the course CSO3 above 2 assigned to each full-time faculty member $f$. *Used for soft constraint 7*

#### Objective Function
Maximize $\displaystyle \sum_{f}\sum_{c} r_{f,c} P_{f,c} S_f - W_1 \sum_{f} {u_f} - W_2 \sum_{f} {v_f}$

#### Constraints
##### Hard Constraints
**Domain constraints**
$r_{f,c} \in \mathbb{Z}_0^+$
$b_{f,c} \in \{0,1\}$
$u_f \in \mathbb{Z}_0^+$
$v_f \in \mathbb{Z}_0^+$

1. **Each faculty is assigned to only the courses that he/she is qualified to teach**

$\displaystyle \forall f, c: r_{f,c}(1-Q_{f,c}) = 0$

2. **Each full-time faculty member’s actual course load must be equal to his/her required course load per semester according to their contracts**

$\displaystyle \forall f \neq \alpha: \sum_{c} r_{f,c} = L_f$

3. **The number of classes assigned to faculties needs to be equal to the number of classes required by each course**

$\displaystyle \forall c: \sum_{f} r_{f,c} = N_c$

4. **Relate $r_{f,c}$ to its boolean counterpart $b_{f,c}$**

$\displaystyle \forall f \neq \alpha, \forall c: r_{f,c} \leq b_{f,c} \cdot L_f$
$\displaystyle \forall c: r_{\alpha,c} \leq b_{\alpha,c} \cdot M$
$\displaystyle \forall f, c: r_{f,c} \geq b_{f,c}$

##### Soft Constraints
5. **All courses, and particularly CSO6, are to be taught by full-time faculty, if possible**

This soft constraint is encoded by defining $P_{\alpha,c}$. Namely:
$P_{\alpha, c} = \begin{cases} -1, & \text{if } c \neq 6 \\
-X, & \text{otherwise}\end{cases}$

We assumed $P_{\alpha, c} = -1$ by default because we reasoned its preference should be lower than any possible explicit preference by the full-time faculty members; since the lowest preference among faculties was $0$, we chose a negative value. For simplicity's sake, we chose $-1$ as a negative number with a small magnitude.

6. **Ideally, the number of preparations (different courses that each faculty taught per semester) for each full-time faculty would be no more than two**

$\displaystyle \forall f \neq \alpha: \sum_{c} b_{f,c} \leq 2 + u_f$

7. **Ideally, no more than two classes of the course CSO3 should be assigned to any full-time faculty**

$\displaystyle \forall f \neq \alpha: r_{f,3} \leq 2 + v_f$

---

### Scheduling Phase
The MWF and TuTh problems have the same formulation, with different input set and duration $\Delta$ for time slots.

#### Sets and Indices
- Time slot IDs ($T$): $\{1,2,3,4,5,6,7\}$

Both MWF and TuTh have seven time slots, and this formalization makes it easier to translate this problem for the solver.

- MWF time slots: $\{\text{9:10}, \text{10:20}, \text{11:30}, \text{13:30}, \text{14:40}, \text{15:50}, \text{17:25}\}$
- TuTh time slots: $\{\text{8:15}, \text{9:50}, \text{11:25}, \text{13:00}, \text{14:35}, \text{16:10}, \text{18:00}\}$

We chose to represent time slots as the number of minutes after midnight that day; for instance, 9:10 AM is represented as $9 \times 60+10=550$. This simplifies the calculations involving time without requiring the implementation of a more complex data type.

#### Parameters
$S_f$: Seniority of faculty member $f$

$R_{f,c}$: Number of classes of course $c$ given by faculty member $f$ (result from the previous stage).

$P_{f,t}$: Preference of faculty member $f$ for time slot $t$

$T_t$: Start time of time slot $t$, in minutes since the beginning of the day (0:00)
$\Delta$: Duration of the time slots, in minutes

$W_3$: Weight that represents how important it is for classes not to start too early (before 9:00 AM) or end too late (after 4:00 PM) in a day (soft constraint 11)
$W_4$: Weight that a represents how important it is for faculty members to not lecture more than 2 consecutive classes (soft constraint 12)
$W_5$: Weight that a represents how important it is for faculty members to not lecture consecutive classes in the afternoon (soft constraint 13)

#### Decision Variables
$a_{f,c,t}$: Assignment of a course-faculty pair $(f,c)$ to a time slot $t$.

$y_{f,t}$: Boolean variable that is true if there are more than two consecutive classes, starting at time slot $t$, for faculty member $f$. *Used for soft constraint 12*

$z_{f,t}$: Boolean variable that is true if there are consecutive classes in the afternoon, starting at time slot $t$, for faculty member $f$. *Used for soft constraint 13*

#### Objective Function
Maximize $\displaystyle \sum_{f}\sum_{c}\sum_{t} a_{f,c,t} P_{f,t} S_f - W_3 U - W_4 \sum_{f}\sum_{t} y_{f,t}S_f - W_5 \sum_{f}\sum_{t} z_{f,t}S_f$

#### Constraints
##### Hard Constraints
**Domain constraints**
$a_{f,c,t} \in \{0, 1\}$
$y_{f,t} \in \{0, 1\}$
$z_{f,t} \in \{0, 1\}$

8. **Each full-time faculty member can be teaching at most one course on a given time slot**

$\displaystyle \forall f \neq \alpha, t: \sum_c a_{f,c,t} \leq 1$

9. **A teacher must lecture exactly as many time slots of a course as he was assigned in a previous stage**

$\displaystyle \forall f, c: \sum_t a_{f,c,t} = R_{f,c}$

We included Constraints 8 and 9 to keep the semantics of the decision variables consistent with the values that they may take.

10. **At most three course classes can be assigned to the same class period**

$\displaystyle \forall t: \sum_{f}\sum_{c} a_{f,c,t} \leq 3$

##### Soft Constraints
11. **Most of the students and faculty prefer that their class day begins after 9:00 AM and ends before 4:00 PM**

$\displaystyle U = \sum_{f} \sum_{c} \sum_{\substack{t: T_t < 9:00\,\vee \\ T_t+\Delta > 16:00}}{a_{f,c,t}}$

*Note: $U$ is a temporary variable used solely in the definition of the objective function*

12. **Avoid having instructors teach more than two consecutive class periods**

$\displaystyle \forall f, \forall t \leq |T| - 2: \sum_{c} (a_{f,c,t} + a_{f,c,t+1} + a_{f,c,t+2}) \leq 2 + y_{f,t}$

13. **Avoid having instructors teach back-to-back classes in the afternoon (after 12:00)**

$\displaystyle \forall f, \forall t \leq  |T| - 1 \wedge T_t \geq \text{12:00}: \sum_{c} (a_{f,c,t} + a_{f,c,t+1}) \leq 1 + z_{f,t}$

## Results

Our solution used the following weights:

$W_1 = 1$
$W_2 = 4$
$W_3 = 10$
$W_4 = 1$
$W_5 = 1$
$X = 2$

We initially set all weights to 1 by default, and then balanced some weights until all soft constraints were reasonably respected. For some constraints (the ones using $W_1$, $W_4$ and $W_5$) did not require changes because they were only used as a tiebreaker in cases where the main part of the objective function (regarding preferences and seniority) was the same. However, we had to increase weights $W_2$ and $W_3$ for the related soft constraints to be taken into account.

### Course Assignment Phase

Objective Function = 446

| Faculty         | CSO1 | CSO2 | CSO3 | CSO4 | CSO5 | CSO6 | CSO7 | CSO8 |
|-----------------|------|------|------|------|------|------|------|------|
| Barbosa         | 1    | 0    | 2    | 0    | 0    | 0    | 0    | 0    |
| Castro          | 0    | 0    | 0    | 1    | 0    | 1    | 0    | 0    |
| Gerardo         | 0    | 0    | 0    | 0    | 0    | 2    | 0    | 1    |
| Lameiras        | 1    | 2    | 0    | 0    | 0    | 0    | 0    | 0    |
| Machado         | 0    | 0    | 1    | 0    | 0    | 0    | 0    | 0    |
| Pedro           | 0    | 1    | 2    | 0    | 0    | 0    | 0    | 0    |
| Queiros         | 0    | 3    | 0    | 0    | 0    | 0    | 0    | 0    |
| Soeiro          | 0    | 0    | 0    | 0    | 0    | 3    | 0    | 0    |
| contingent      | 0    | 0    | 0    | 0    | 1    | 0    | 1    | 0    |

### Scheduling Phase

#### MWF

Objective Function = 210

| Time | Classes |
|-|-|
| 09:10 | (contingent - CSO7) |
| 10:20 | (Barbosa - CSO3), (Pedro - CSO3), (Queiros - CSO2) |
| 11:30 | (Pedro - CSO2), (Queiros - CSO2) |
| 13:30 | (Barbosa - CSO3), (Pedro - CSO3), (Queiros - CSO2) |
| 14:40 | (Barbosa - CSO1), (contingent - CSO5) |
| 15:50 | (Castro - CSO6) |
| 17:25 | (Castro - CSO4) |

#### TuTh

Objective Function = 182

| Time | Classes |
|-|-|
| 08:15 | (Machado - CSO3) |
| 09:50 | (Gerardo - CSO6), (Lameiras - CSO2), (Soeiro - CSO6) |
| 11:25 | (Gerardo - CSO6), (Lameiras - CSO2), (Soeiro - CSO6) |
| 13:00 | (Lameiras - CSO1) |
| 14:35 | (Gerardo - CSO8), (Soeiro - CSO6) |
| 16:10 |  |
| 18:00 |  |

## Discussion

### Results

#### Course Assignment

We were able to maximize the course preferences for all faculties and obtain a solution that also respects the three relevant soft constraints.

#### Scheduling

We adjusted the previously mentioned weights until we obtained a solution that prioritized the following topics, in this order:
- Respecting faculty preferences
- Avoiding classes that start before 9:00 AM or end after 4:00 PM
- Avoiding consecutive classes

The final outcome ended up being a satisfactory balance of these aspects, in that only three classes were outside the ideal 9:00AM-4:00PM period and only two professors didn't get their preferred time slot preferences (instead being assigned their second choice). As for the various occasions of consecutive classes, pretty much all of them happened because there was no other way to fulfill the teacher's preferred time slot preferences.

### How to handle seniority

We initially used seniority as an additive factor to the score of an allocation, meaning that, for the course assignment stage, the score for an assignment $r_{f,c}$ was equal to $r_{f,c} (A \cdot P_{f,c} + B \cdot S_f)$, and, for the scheduling stage, the score for an assignment $a_{f,c,t}$ was $a_{f,c,t} (C \cdot P_{f,t} + D \cdot S_f)$. We then realised this formulation did not respect the intended interpretation of seniority, as by separating the first component in the course assignment stage we would end up with $r_{f,c} \cdot A \cdot P_{f,c}$, which meant that faculty preferences were not being valued according to their seniority. We believe that multiplying faculty preferences by seniority is a more appropriate way to interpret seniority.

In addition, we also used seniority as a multiplier for the penalty variables $y_{f,t}$ and $z_{f,t}$ (which relate to soft constraints regarding consecutive classes). This means that faculties with a higher seniority will have a smaller likelihood of being assigned consecutive classes.

### Hard and soft constraints

The role of hard and soft constraints in our formulations follows expectations: hard constraints must be enforced in order for the solution to be valid, whereas soft constraints are incorporated as penalties within the objective function. For each soft constraint, we devised a way to quantify how much a given solution violates that soft constraint, and then multiplied that score by a weight parameter to be able to vary the degree to which it affected the objective function.

### Alternative solution

We believe that the steps suggested by the statement in order to split the problem into two, and then further split the second problem's input into two sets, could in reality be ignored, since despite both problems having complex formulations, neither had terrible performance when being solved by CPLEX. Therefore, we could achieve an even more optimized solution. We also believe that the potential improvements from this approach would be more substantial in cases where the contingent is responsible for a greater number of classes.

The reformulation should be somewhat simple, as we only need to use the parameters and decision variables of the two stages and combine their objective functions.
