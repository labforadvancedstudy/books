# Making Choices (L0)

## The Algorithm of Life

Before we dive into computational algorithms, let's start with the most fundamental algorithm we all run every day: making choices. This is L0 - the ground level where abstract computer science meets the messy reality of human existence.

## Every Day is an Algorithm

From the moment you wake up, you're executing decision algorithms:

```
morning_routine():
    if alarm_rings():
        if feeling_tired():
            hit_snooze()  # Greedy algorithm: optimize for immediate comfort
        else:
            get_up()
    
    while not ready_for_day:
        make_choice([shower, coffee, breakfast, check_phone])
```

## The Hidden Complexity

What seems simple is actually incredibly complex:
- **Input**: Current state, goals, constraints
- **Processing**: Weigh options, predict outcomes
- **Output**: A decision
- **Feedback**: Learn from results

## Types of Everyday Algorithms

1. **Greedy Choices**: "What do I want most right now?"
   - Eating the dessert first
   - Buying that thing you don't need
   - Watching one more episode

2. **Optimal Stopping**: "When do I stop looking?"
   - Apartment hunting (37% rule)
   - Dating (explore then commit)
   - Job searching

3. **Scheduling**: "What order should I do things?"
   - Shortest task first (clear small todos)
   - Deadline-based (do urgent things)
   - Energy-based (hard tasks when fresh)

4. **Caching**: "What should I remember?"
   - Where you left your keys
   - People's names
   - Frequently used passwords

## Real Algorithms You Already Use

```python
# The "What to Wear" Algorithm
def choose_outfit(weather, occasion, clean_clothes):
    if occasion == "formal":
        return clean_clothes.filter(formal=True).pick_best()
    elif weather.is_cold:
        return layer_up(clean_clothes)
    else:
        return clean_clothes.pick_comfortable()

# The "What to Eat" Algorithm  
def decide_dinner(energy, time, ingredients):
    if energy < 20 and time < 30:
        return order_delivery()
    elif ingredients.is_empty():
        return go_shopping()
    else:
        return cook(ingredients.pick_easiest())
```

## The Paradox of Choice

More options don't always lead to better outcomes:
- **Analysis Paralysis**: Too many choices freeze us
- **Decision Fatigue**: Each choice depletes energy
- **Regret**: Wondering about unchosen paths
- **Satisficing vs Maximizing**: Good enough vs perfect

## Heuristics: Mental Shortcuts

Our brains use approximate algorithms:
1. **Availability**: Judge by what comes to mind easily
2. **Anchoring**: First information biases everything
3. **Recognition**: Choose what seems familiar
4. **Social Proof**: Do what others do

## When Human Algorithms Fail

- **Sunk Cost Fallacy**: Considering past investment in future decisions
- **Confirmation Bias**: Only seeing evidence that supports our choice
- **Present Bias**: Overvaluing immediate rewards
- **Probability Blindness**: Bad at estimating unlikely events

## Improving Your Decision Algorithms

1. **Add Randomness**: Sometimes flip a coin to break ties
2. **Set Constraints**: Limit options to simplify choices
3. **Batch Process**: Make similar decisions together
4. **Create Defaults**: Pre-decide routine choices
5. **Time Box**: Set deadlines for decisions

## The Meta-Algorithm

```python
def make_better_choices():
    while alive:
        decision = face_choice()
        
        if decision.is_reversible:
            just_try_something()
        elif decision.is_important:
            sleep_on_it()
        else:
            go_with_gut()
        
        learn_from_outcome()
```

## Connection to Computer Science

Every computational concept has a life parallel:
- **Sorting**: Prioritizing tasks
- **Searching**: Finding lost items
- **Optimization**: Making life better
- **Recursion**: Habits that reinforce themselves
- **Parallel Processing**: Multitasking
- **Caching**: Memory and learning

## The Beautiful Truth

You've been doing computer science your whole life. Every choice is an algorithm, every day is a program, every life is a computation. Understanding formal algorithms just gives names and rigor to what you already intuitively know.

## Connection to Other Concepts

- **Algorithm** (L1): Formal version of choice-making
- **Greedy** (L5): How we often choose
- **Optimization** (L4): Making better choices
- **Halting Problem** (L8): Some life decisions never resolve