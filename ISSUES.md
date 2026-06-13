# Issues with generated courses

## Version 2

### Changes

Classical games included.

No mid-run shrinking of scope; it stays fixed.

Strict coverage — every reply with ≥4% of games is evaluated, period. The real culprit behind 6...Nbd7 and 7...Nc6 was twofold — the old 10% threshold and a per-ply children cap that tapered to 1 past move 12. Both are gone; branching depth is now controlled by the data (explorer games thinning below MIN_GAMES_TO_EXPAND), with only an 8-candidate safety net.

Transpositions — the builder remembers every expanded position by EPD. A repeat arrival gets a transposes_to pointer plus the canonical line's White reply (cloned, so the same position always gets the same repertoire move), and the report shows [→ transposes: …] so authoring can comment both directions.

&Line endings — a line can no longer end while White is down more than 1 pawn of material (sacrifices play out until the advantage is on the board), while Black's best reply is forced (≥150cp gap to the second choice), or while a forced mate is pending — mates are now played to checkmate.

Engine lines are cached; claims in comments, especially tactical claims, are checked post-writing.

### Observations

The Stockfish run to get the data (cached, thankfully) for this one took a solid day. A bigger opening would require more hardware or more patience.

Chess language - 'exploit the light-colored squares', 'attack', etc. - has a chance of being flagged by Claude Fable's overaggressive cybersecurity filter. This broke a run, and a lot of work got done on Opus 4.8 and had to be thrown out.

### Output

1 PGN file, 131 annotated games, organized across 16 chapters.

### Issues

General impression - I feel like the later chapters have more weaknesses than the earlier ones. Length of context window issue? I may try chunking the work and assigning it to sub-agents.

Statement evaluation catches mistakes, but it needs to be more granular and cover every claim - some minor inaccuracies slip through.

The transpositions could be covered with comments on parent positions, instead of being given their own separate trained lines.

There is no explanation why attractive looking moves for white are in fact bad; this is because the engine gets top lines, and doesn't look at bad ones. I may have the engine look at all player moves over 4%, not just the one chosen by the course, and pull the engine lines for each, so the PGN can include info on traps for the player to avoid.

### To Do

Re-watch some of William Graif's Alapin Gambit videos, to look for differences.

## Version 1

### Prompt used

I have found that you possess strong chess understanding when you supplement your analyses with Stockfish evaluations. I would like you to develop a course for Checkable by using the lichess API to gather data - I'm thinking mainly of the opening explorer API - as well as Stockfish as needed to evaluate recommended moves. For the opening I specify, you should look at popular moves at all levels, making sure to include lines for common mistakes at various levels as well as the best moves my opponent can make. (Please also use Stockfish to check these moves, and provide evaluations.)

I would also like you to provide your strong chess understanding to provide comments on the moves - why are they made? What are their strengths?

You should probably store all necessary data for this course in a subdirectory under a courses directory. The end product should include a way to import the course into Checkable, so anyone with the contents of the course subdirectory and a local copy of Checkable could do it. (You may have to create the import functionality in Checkable itself.)

To start this test, I would like you to create a course for White on the Alapin Gambit, starting with 1 e4 e6 2 d4 d5 3 Be3, inviting black to play 3...dxe4. All lines must start with 1 e4 e6 2 d4 d5 3 Be3.

### Follow-up questions

Asked by Claude Fable, answered by me.

Q1) How deep and broad should the course be? Deeper lines mean more Lichess API calls and Stockfish time, but a more complete repertoire.

A1) Deep (~14-16 moves, 50+ lines)

Q2) Which Lichess opening explorer pool should drive 'popular moves and common mistakes'? (I can blend more than one.)

A2) All levels, blitz+rapid

Q3) How much Stockfish time per position? This sets the quality bar for the evaluations shown in annotations.

A3) Depth ~24

### Output

1 PGN file, 83 annotated games, organized in six chapters

### Issues

1) Generator used blitz + rapid games. There was no reason to not include classical games. This was my oversight.

2) The further the course progressed, the shorter the lines. I believe this was due to Claude Fable adjusting dynamically, based on the length of time required to produce the course - Stockfish being the big delay.

This isn't great - I'd rather do faster analysis at lower depth but cover sufficient nodes, and I don't care how long the overall run takes, whether it's hours or days.

(Also, can multiple local instances of Stockfish be run in parallel? Or is Stockfish already optimized enough that that'd defeat the point?)

3) In the position 1 e4 e6 2 d4 d5 3 Be3 dxe4 4 Nd2 Nf6 5 f3 exf3 6 Ngxf3, 6...Nbd7 was not covered - perhaps because it was Black's 6th most popular move. However, it occurs 5% of the time and is one of the better ones. We should be strict about covering anything that happens >4% of the time by default, with this parameter being adjustable.

4) In the position 1 e4 e6 2 d4 d5 3 Be3 dxe4 4 Nd2 Nf6 5 f3 exf3 6 Ngxf3 Be7 7 Bd3, 7...Nc6 was not covered. This is Black's 3rd most popular move. I'll pause mentioning omitted moves from here, since this is the same issue as 3) and should be caught by the same rule. (In short, I believe we over-curated.)

5) Transpositions should be mentioned in comments. For example:
- In the position 1 e4 e6 2 d4 d5 3 Be3 dxe4 4 Nd2 Nf6 5 f3 exf3 6 Ngxf3 Nbd7 7 Bd3, 7...Be7 is the 2nd most common move.
- In the position 1 e4 e6 2 d4 d5 3 Be3 dxe4 4 Nd2 Nf6 5 f3 exf3 6 Ngxf3 Be7 7 Bd3, 7...Nbd7 is the 5th most common move.

I would expect one of these two to start a line - probably 1 e4 e6 2 d4 d5 3 Be3 dxe4 4 Nd2 Nf6 5 f3 exf3 6 Ngxf3 Nbd7 7 Bd3 Be7. I would also expect the comments on 1 e4 e6 2 d4 d5 3 Be3 dxe4 4 Nd2 Nf6 5 f3 exf3 6 Ngxf3 Be7 7 Bd3 to mention that 7...Nbd2 here transposes to 6...Nbd7 7 Bd3 Be7, covered elsewhere.

6) At times lines end prematurely, even when long. In particular, if in a situation where the player has just sacrificed material, show the continuation until the advantage is clear.

For example, the line ending with 13. Rxf7 (sacrificing a rook for a pawn!) and the comment 'The half-open f-file cashes in: +3.3. The rook cannot be accepted comfortably, and f7 simply collapses.' is great, but it needs to be shown. Why not 13...Rxf7 14 Nxf7
Kxf7? (It's mate in 3, that's why.)

7) Don't end lines when there's a series of only-moves to make. If black must make one specific move to fend off disaster, and white only has one good move in response, keep going.

8) When selecting a choice that is not engine-best, the coursegen README.md states "White-move selection prefers the most popular explorer move within 25cp of the engine's best — practical chess over engine-first chess." (This should be 'player-move selection', not 'white-move selection'.) I think this is fine as long as the most popular move also has good winning chances, but that should be noted.

9) Full engine lines beyond the end of a line should be kept and considered when annotating, to ensure bold claims are accurate.

For instance, Chapter 6 claims that 1 e4 e6 2 d4 d5 3 Be3 c5 4 dxc5 Qa5+ 5 Nc3 Bxc5 6 Bb5+ wins a piece. It's a great line (6 Bb5+ is played only 10% of the time) and a *fantastic* position, *equivalent* to being a piece up, but it does not win a piece - 6...Bd7 7 Bxc5 Bxb5 reclaims it. (Of course 6...Nc6 7 Bxc5 *does* win a piece and that's worth mentioning.)

This line ended too soon anyway - it was late in the run - but saving and utilizing the engine lines would've caught this.
