# Frequently Asked Questions about Leela Zero #

**Note about Chinese translation**
Earlier versions of this FAQ list were in both English and Chinese. Unfortunately, the FAQs became rather out of date. The old FAQs, in both English and Chinese, are reproduced at the end of this page.

## What is Leela Zero?

Leela Zero is a program that plays the ancient game of go. It uses [Monte Carlo tree search] (https://en.wikipedia.org/wiki/Monte_Carlo_tree_search), with a neural network (trained on self-play games) guiding the search and providing evaluation at the leaves. The network design, search strategy, and training process are closely modelled on Google's [AlphaGo Zero project](https://deepmind.com/blog/alphago-zero-learning-scratch/).

Lacking Google's vast computational resources, Leela Zero's self-play games are provided by a large number of volunteers.

## Who made it?

Gian-Carlo Pascutto. He is also the author of an earlier MCTS-plus-neural-network go program, called just [Leela](https://sjeng.org/leela.html); after seeing the success of AlphaGo Zero, GCP launched the Leela Zero project. (Hence its slightly odd name.) Leela Zero is considerably stronger than Leela.

## How does it work?

At its heart is a deep convolutional neural network, which takes as input a Go position and outputs two things: first, an assessment of how likely each player is to win the game from that position (the position's "value"); second, a score for each legal move indicating how plausible a next move it is (this is called the "policy").

In order to choose a move, the MCTS algorithm constructs a search tree as follows. Initially, the tree has just one node, the initial position in which we have to move. It will gradually grow by repeatedly adding new positions reachable by moving from positions already in the tree. For each position in the tree, we keep track of how many times we have looked at it, and the average of all the values we assigned it when we did.

Now, we repeatedly do this: Start at the root of the tree. Find the "most interesting" move, in a sense I'll describe in a moment. If the resulting position is already in the tree, proceed to find the "most interesting" move from there, and continue until we reach a position that isn't already in the tree. Add it to the tree, and record the value given to it by the neural network. Now work back up the tree, incorporating this value into the averages of the nodes we traversed.

We do this either a fixed number of times, or until a certain amount of time has elapsed. Then the move we choose is the one we have visited most often during this process. (More or less. Early in the game, we pick a move at random, with probability proportional to how often it's been visited, so that our games aren't all the same. And in self-play training games we introduce further randomness for the sake of exploring a wider range of possible positions.)

The search process depends crucially on that notion of "most interesting" move. The interestingness of a move from position P to position Q is the sum of two terms. The first is simply the estimated winning probability in position Q (a move is interesting if it is to a position we think is good). If we haven't evaluated Q with the neural network yet, we use the estimated winning probability at P (which we always _do_ have), minus a correction that depends on how promising-according-to-policy the _other_ children of P that we _have_ already looked at are. The second is proportional to the neural network's "policy" value for the move from P to Q (a move is interesting if the neural network thinks it's promising), inversely proportional to the number of times we've already looked at Q (a move is interesting if we don't already know a lot about its consequences), and proportional to the square root of the number of times we've already looked at P (the more we already understand P, the more eager we are to try new things there rather than searching more deeply in children we've already looked at).

For information about the structure of the neural network and how it is trained, have a look at what Google wrote about it at <https://deepmind.com/blog/alphago-zero-learning-scratch/>.

## How well does it play?

Leela Zero is constantly improving, so any very specific answer is liable to be out of date. However, here are a few comparisons that may be helpful.

In mid-2018, retired professional Hajin Lee 4p ("Haylee") played a series of 8 games ([first game on YouTube](https://www.youtube.com/watch?v=1buJ9y7dwU8); [last game on YouTube](https://www.youtube.com/watch?v=lRXjP9ZbA2Q)) against Leela Zero, at a range of handicaps. In this match, Leela Zero won comfortably in games where it gave a handicap no bigger than 2 stones, and lost badly in games where it gave a handicap of 3 stones. (In these games, white had the usual komi of 7.5 points.)

In mid-2018, a version of Leela Zero using a larger neural network trained from the same self-play games as the "official" Leela Zero networks entered the Tencent World AI Weiqi Competition. As of the writing of this FAQ, only the first stage of this competition has been held; Leela Zero is in second place after Tencent's _Fine Art_. (But this larger network is not yet publicly available, and "official" Leela Zero is definitely weaker.)

Leela Zero is not the strongest publicly available program (as of mid-2018). In particular, Facebook's _ELF OpenGo_ -- another project modelled closely on AlphaGo Zero -- is stronger than any official version of Leela Zero to date, although in the Tencent competition it lost to an unofficial larger-network version of Leela Zero.

## Why is it weaker than AlphaGo?

For three reasons. First, the Leela Zero community's resources are much less than Google's. The strongest versions of AlphaGo have been trained with many more self-play games than Leela Zero has. However, this is probably not the whole story; Google reports that AlphaGo Zero was Google reports that AlphaGo Zero surpassed the version of AlphaGo that beat Lee Sedol after about 5 million self-play games, and Leela Zero was definitely not that strong after playing 5 million games.

Second, Leela Zero uses a smaller neural network than AlphaGo Zero.

Third, Leela Zero runs on normal PC hardware whereas AlphaGo Zero uses special-purpose hardware designed by Google for accelerating neural network computations.

## Why does Leela Zero's neural network use only 15 blocks, when AlphaGo uses 40?

(Leela Zero may move to larger networks over time, but it is likely to stay well below 40 blocks for a good while.)

Leela Zero is a community project; both for self-play training and for actual use, it needs to run on "ordinary" computing hardware not equipped with Google's TPUs.

The early stages of the Leela Zero project used even smaller networks, and were able to make faster progress by doing so. The general strategy has been to advance to larger networks only when learning appears to have stalled, suggesting that the maximum capabilities of the network being used may have been reached.

## Can I play against Leela Zero?

Yes, in at least two different ways. There are Leela-Zero-based bots on all the usual go servers; and you can download Leela Zero and run it on your own computer.

Leela Zero will run happily on Windows, macOS or Linux. Ready-built executables are available for Windows (<https://github.com/gcp/leela-zero/releases>); if you're running Linux or macOS, you will have to download the source code (available from the same location) and build it yourself.

## What hardware do I need to run Leela Zero?

Any sort of PC (Mac included) will do. Leela Zero, like any program based on a neural network, will perform *much* better if you have a GPU, and better still if you have a nice fast one.

## What software do I need to play against Leela Zero?

Obviously you need Leela Zero itself. On top of that, you will want something that can display a Go board, allow you to play moves by clicking on it, and so forth. For that, any GUI that knows how to use Go-playing engines that speak the "Go Text Protocol" (or GTP for short) will do. One commonly used program is [Sabaki](https://sabaki.yichuanshen.de/).

Another option is [Lizzie](https://github.com/featurecat/lizzie), a program developed specifically to be a user interface for Leela Zero. It can show you what Leela Zero is thinking about, its estimated win probabilities for candidate moves, and so forth. As of mid-2018 its user interface is less polished than Sabaki's, but its LZ-specific features are very useful.

## When I run Sabaki/Lizzie/... for the first time, what's this "auto-tuning" thing that takes so long?

Leela Zero is trying many slightly different ways to organize its calculations to run as quickly as possible on your GPU.

## Most test matches between older and newer networks "fail". Why doesn't learning make progress?

First of all, a "fail" result need not mean that the new network is weaker. A network is promoted according to a criterion called SPRT (<https://en.wikipedia.org/wiki/Sequential_probability_ratio_test>), whose idea is to promote networks only when we're quite confident that they are better, so that if you switch to a new network you can be reasonably confident that LZ will genuinely get stronger as a result. The precise criterion is something like this: we consider the hypotheses "new network will win 55% or more of games against old network, in the long run" and "new and old networks are equally strong"; initially we suppose these are equally likely; we play a match and stop when the results we've seen make us 95% confident about which hypothesis is correct, or when we have played 400 games without that happening.

Second, even when real learning progress is happening it doesn't always show up immediately in improved results. Consider the old proverb "learn joseki, lose two stones".

Third, neural network learning is a stochastic process and it is not guaranteed always to be making forward progress. Maybe some day someone will come up with a training method that never goes backwards, but for the moment the best training methods sometimes do.

## I looked at some of the self-play games and they were full of terrible moves. What gives?

Some of the terrible moves are deliberate. In order for Leela Zero to learn how to play well in a wide variety of positions, the self-play process deliberately makes some moves that are not best, thus exploring a wider range of positions.

Some of them are a consequence of wanting to generate self-play games quickly. They are played using a limited number of "visits", meaning that the search examines only a certain number of positions; if that number were increased, the gain from having those games be better played might be outweighed by having fewer of them. If you play a serious game with Leela Zero, it will most likely look at 10-100 times more positions than in its training games.

If you are looking at the later stages of the game then you will likely see some peculiar moves. Leela Zero's assessment of a position is based on how likely it thinks each player is to win; it doesn't care about the difference between winning by half a point and winning by a hundred points, except in so far as a position with a 100-point advantage is harder to mess up. If the game is not very close, Leela Zero may make moves that lose points but (in its judgement) have negligible impact on the outcome.

## How can I play a game against Leela Zero without just getting crushed?

Unless you are a top professional (and perhaps, by the time you are reading this, even if you are -- Leela Zero is constantly getting stronger) you will have little chance of beating Leela Zero in a game without handicap.

Unfortunately, if Leela Zero gives you a large handicap, it is likely to play badly (because every position it considers has close-to-zero chance of winning). If you're a dan-level player then you may find that if Leela Zero gives you a large enough handicap for you to have a chance, this makes it play so badly that _it_ doesn't have a chance.

This is a tricky problem, being worked on actively by the Leela Zero community. As of mid-2018, though, it is an unsolved problem.

(If you are _not_ a dan-level player, though, just take as many stones of handicap as you need. Even with its difficulty playing against a handicap, Leela Zero will probably still be much stronger than you.)

## The final score is wrong!

Leela Zero uses [Tromp-Taylor rules](https://senseis.xmp.net/?TrompTaylorRules), which are simple and elegant but have the unfortunate property that if you want dead groups not to count against the territory they are inside then you must actually capture them. (Because Tromp-Taylor is a "Chinese" system in which stones on the board count toward the score of the player who owns them, doing this doesn't attract any penalty. But players will not always bother to do it, when it doesn't affect the result.)

## There's a match shown on the home page whose winner didn't get promoted. Why?

It's probably a _test match_ between the currently-best Leela Zero network and something else such as Facebook's "Elf OpenGo", or an experimental network with a different architecture, or a network trained from human games.

These matches are played to measure Leela Zero's progress, to get some idea of whether moving to a larger network might be helpful, or just out of curiosity.

## I want to contribute to the Leela Zero project. How can I do that?

You can allow your computer to be used for self-play games. To do this, download Leela Zero and run the `autogtp` program included with it. It will contact the server used for coordinating Leela Zero self-play and start playing games against itself. To stop this, just kill autogtp (e.g., by hitting control-C in its window). You can run it again later.

You can get involved in the development of the software. Good first steps would be to look at the [GitHub issue tracker](https://github.com/gcp/leela-zero/issues), and to visit the [Discord server](https://discordapp.com/channels/417022162348802048) and discuss whatever ideas you may have.

If you really want to, you can surely arrange to contribute money or hardware to the project, but it is not actively soliciting donations.

----

**Older FAQs (see note at top)**

# Leela Zero常见问题解答 #
# Frequently Asked Questions about Leela Zero #

## 为什么网络不是每次都变强的 ##
## Why doesn't the network get stronger every time ##

从谷歌的论文中可以发现，AZ的网络强度也是有起伏的。而且现在只是在小规模测试阶段，发现问题也是很正常的。请保持耐心。

AZ also had this behavior, besides we're testing our approach right now. Please be patient.

## 为什么比较两个网络强弱时经常下十几盘就不下了 ##
## Why only dozens of games are played when comparing two networks ##

这里使用的是概率学意义上强弱，具体来说是SPRT在95%概率下任何一方有超过55%的胜率（ELO的35分），就认为有一方胜出了。谷歌的论文中是下满400盘的。唯一的区别是我们这里的Elo可能不是那么准确，网络的强弱还是可以确定的。

We use SPRT to decide if a newly trained network is better. A better network is only chosen if SPRT finds it's 95% confident that the new network has a 55% (boils down to 35 elo) win rate over the previous best network.

## 自对弈时产生的棋谱为什么下得很糟 ##
## Why the game generated during self-play contains quite a few bad moves ##

生成自对弈棋谱时，使用的MCTS模拟次数只有3200，还加入了噪声，这是为了增加随机性，之后的训练才有进步的空间。如果用图形界面（如sabiki）加载Leela Zero，并设置好参数与之对弈，你会发现它其实表现得并不赖。

The MCTS playouts of self-play games is only 3200, and with noise added (For randomness of each move thus training has something to learn from). If you load Leela Zero with Sabaki, you'll probably find it is actually not that weak.

## 有些自对弈对局非常短 ##
## Very short self-play games ends with White win?! ##

自对弈的增加了随机性，一旦黑棋在开始阶段选择pass，由于贴目的关系，白棋有很大概率也选择pass获胜。短对局由此产生。

This is expected. Due to randomness of self-play games, once Black choose to pass at the beginning, there is a big chance for White to pass too (7.5 komi advantage for White). See issue #198 for defailed explanation.

## 对局结果错误 ##
## Wrong score? ##

Leela Zero使用Tromp-Taylor规则(详见<https://senseis.xmp.net/?TrompTaylorRules>)。虽然与中国规则一样贴7.5目，但为计算方便，并不去除死子。因此，结果与使用中国规则计算可能有所不同。不过，不去除死子并不影响模型的训练结果，因为双方会将死子自行提掉。

Leela Zero uses Tromp-Taylor rules (see https://senseis.xmp.net/?TrompTaylorRules). Although its komi is 7.5 as in Chinese rule, for simplicity, Tromp-Taylor rules do not remove dead stones. Thus, the result may be different from that calcuated using Chinese rule. However, keeping dead stones does not affect training results because both players are expected to capture dead stones themselves.

