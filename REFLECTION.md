# PA2 Reflection

## 1. What I wrote for each TODO

For these time complexities, n is the number of scenes. I’m mostly counting how many scenes the code visits. I mention the extra cost of copying path strings at the end.

1. **Scene constructor (TODO 1):** I make an array with exactly as many slots as the scene has choices, then set each slot to `nullptr`. If it is an ending, there is no array. Since a scene can have at most three choices, this is O(1).

2. **Scene destructor (TODO 2):** I delete the child scenes first, then the choices array, and finally the scene’s data. The children delete their own children too, so deleting a whole subtree takes O(n).

3. **isEnding (TODO 3):** This just checks whether the number of choices is zero. It is O(1).

4. **getChoice (TODO 4):** I check the index before reading the slot. An invalid index gives back `nullptr`; otherwise I return that child. It is O(1).

5. **setChoice (TODO 5):** If the index is valid, I put the new scene pointer in that slot. If it is invalid, nothing happens. It is O(1).

6. **countScenes (TODO 6):** I count the current scene as one, then recursively add the number of scenes under each choice. A null scene counts as zero. It visits every scene once, so it is O(n).

7. **countEndings (TODO 7):** An ending contributes one to the count. Otherwise, I add up the ending counts from its children. This is O(n).

8. **height (TODO 8):** I find the tallest child subtree and add one for the choice needed to reach it. An ending has height zero, and an empty tree has height -1. This is O(n).

9. **clear (TODO 9):** I delete the root and set it to `nullptr`. Since deleting the root also deletes its descendants, clearing the tree is O(n).

10. **load (TODO 10):** I open the story file, read its starting HP, build the root with `readScene`, and put the finished story in `tree_`. If any step fails, I return false. For a valid file with n scenes, this is O(n).

11. **readScene (TODO 11):** I read one scene’s fields and then call `readScene` again for each of its choices. I also reject a scene with more than three choices. If reading a child fails, deleting the scene cleans up the children already attached to it. This takes O(n) for a subtree with n scenes.

12. **askChoice (TODO 12):** I keep reading a full line until the player enters a valid choice. If input ends, I return zero. Each try is O(1) compared with the number of scenes, but the total depends on how many invalid answers the player enters.

13. **play (TODO 13):** I move through the chosen scenes, updating HP and loot and checking whether an item is already in the bag. The linked-list search can get slower as the bag grows, so the worst case is O(n²) if a long path gives the player many different items.

14. **printStats (TODO 14):** I call the tree’s scene count, ending count, and height functions and print their answers. They each traverse the tree, so together this is still O(n).

15. **printSpoilers (TODO 15):** This starts `printEndings` at the root with an empty path. The traversal visits O(n) scenes.

16. **printEndings (TODO 16):** I recursively go through choice 1, then choice 2, then choice 3, printing a path whenever I reach an ending. It visits O(n) scenes.

17. **greedyRun (TODO 17):** At each scene, I pick the child with the most loot in that *next* scene. I used `>` rather than `>=`, so a tie stays with the earlier choice. It follows one path and visits at most n scenes, giving O(n) scene visits.

18. **bestLoot (TODO 18):** I first check whether entering the scene would kill the player. If not, I recursively check its choices and keep the best surviving loot total and path. Every scene is checked once, so it makes O(n) scene visits.

19. **optimalRun (TODO 19):** I call `bestLoot` from the root, then print the winning path or say that no path survives. The tree traversal is O(n).

The path functions also copy strings such as `"2-1-2-1-2"` as they go. On a very deep tree, those copies add time; the O(n) figures above describe the number of scenes visited.

## 2. Why use exactly `numChoices_` slots?

A scene with one choice only needs one pointer slot. Giving every scene three slots would waste space, especially for endings, which have no children at all. In my constructor, an ending leaves `choices_` as `nullptr`. I also set every allocated slot to `nullptr` right away. That matters if the file fails partway through loading, because the destructor can safely delete the scene even when some choices have not been filled yet.

## 3. What my destructor deletes

I go through the choices and delete each child first. That works recursively: each child frees the scenes below it. Then I free the choices array with `delete[]` and delete `data_`. If I freed the array first, the pointers to the children would be gone while the child scenes were still allocated. Trying to read those slots afterward would also mean accessing freed memory.

## 4. Greedy and optimal in Moonlight Miles

Greedy takes path `1-1`. The rich student’s ride offers 8 loot immediately, so it looks better than investigating with Ranbir. Then greedy takes the shortcut for another 4 loot. It ends up with 12 loot, but Alia’s HP drops to -1 and the run ends in a game over.

Optimal takes `2-1-2-1-2`. Alia goes with Ranbir, checks the cameras, follows the cart, enters the hatch, and records the confession. That route finishes with 14 loot and 7 HP. I made the expensive ride tempting at first, but the safer investigation pays off more if the advisor looks ahead.

## 5. Why checking every path is O(n) scene visits

Checking every possible ending sounds like it might be exponential, but the story is a tree. Each scene has one parent, so `bestLoot` reaches that scene through only one recursive call. It checks every branch without repeatedly solving the same subtree. That means n scenes are visited once each. My path strings do get copied during those calls, which adds work on long paths, but the tree traversal itself is O(n).

## 6. How I use the List

During `play`, I use `bag.search(found)` to check whether Alia already has an item with that name. If she does not, `bag.addFront` adds it. At the end, `bag.size()` gives the inventory count and `bag.print()` lists the items. The list also cleans itself up when `play` ends. A search may have to check all b items in the bag, so one search is O(b). Adding at the front and getting the size are O(1).