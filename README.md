[index.html.html](https://github.com/user-attachments/files/26040027/index.html.html)
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>March Madness 2026 — Prediction Engine v3</title>
<meta name="description" content="8-factor prediction model for the 2026 NCAA Tournament. Spread picks, 1H O/U, sharp money tracking, and methodology.">
<link rel="preconnect" href="https://fonts.googleapis.com">
<link href="https://fonts.googleapis.com/css2?family=Source+Serif+4:opsz,wght@8..60,300;8..60,400;8..60,500;8..60,600;8..60,700&family=DM+Sans:wght@400;500;600;700&family=IBM+Plex+Mono:wght@400;500;600&display=swap" rel="stylesheet">
<script src="https://cdnjs.cloudflare.com/ajax/libs/react/18.2.0/umd/react.production.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/react-dom/18.2.0/umd/react-dom.production.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/babel-standalone/7.23.9/babel.min.js"></script>
<style>
*{box-sizing:border-box;margin:0;padding:0}
::selection{background:#E8D5A0}
body{background:#FAFAF7;font-family:'Source Serif 4',Georgia,serif;color:#2C2C2C}
.card{transition:box-shadow .2s,transform .15s;cursor:pointer}
.card:hover{box-shadow:0 2px 16px rgba(0,0,0,.06);transform:translateY(-1px)}
.nb{cursor:pointer;border:none;background:0;font-family:'DM Sans',sans-serif;font-weight:600;font-size:13px;padding:12px 20px;color:#999;border-bottom:2px solid transparent;transition:.15s;letter-spacing:.3px}
.nb:hover{color:#555}.nb.on{color:#2C2C2C;border-bottom-color:#B8860B}
.pl{cursor:pointer;border:1px solid #E0DDD5;padding:6px 14px;font-family:'DM Sans',sans-serif;font-size:12px;font-weight:500;border-radius:20px;background:#fff;color:#888;transition:.15s}
.pl:hover{border-color:#bbb}.pl.on{background:#2C2C2C;color:#FAFAF7;border-color:#2C2C2C}
@media(max-width:640px){.stats-grid{grid-template-columns:repeat(2,1fr)!important}}
</style>
</head>
<body>
<div id="root"></div>
<script type="text/babel">
const {useState} = React;

/* ══════════════════════════════════════════════════════════════
   DATA — Last updated: March 16, 2026 11:59 PM ET
   To update: replace the data arrays below with fresh lines,
   then re-deploy this single file.
   ══════════════════════════════════════════════════════════════ */

const MW=[{f:"KenPom AdjEM",w:22,c:"#4A90D9"},{f:"Line Movement",w:15,c:"#9B59B6"},{f:"Injuries",w:15,c:"#E74C3C"},{f:"Recent Form",w:13,c:"#27AE60"},{f:"Seed ATS History",w:12,c:"#F5A623"},{f:"BPI / Cross-Model",w:10,c:"#3498DB"},{f:"Coaching",w:8,c:"#E67E22"},{f:"Situational",w:5,c:"#1ABC9C"}];

const G=[
{r:"F4",fv:"UMBC",dg:"Howard",sd:"16v16",open:-1.5,cur:-2.5,tot:140.5,bpi:1.5,sim:3.2,pk:"UMBC -2.5",c:57,t:"LEAN",e:"UMBC 19-2 ATS when favored, covered 15 of 17. Line moved from -1.5 to -2.5 — public on UMBC. Still the right side despite thicker juice.",fx:"Form ↑ · Market ="},
{r:"F4",fv:"NC State",dg:"Texas",sd:"11v11",open:-1.5,cur:-1.5,tot:158.5,bpi:0.1,sim:-1.8,pk:"Texas +1.5",c:53,t:"LEAN",e:"BPI says dead even (NCS by 0.1). Texas H2H win. Line hasn't moved — no sharp signal. Pure coin flip, slight lean on dog getting points.",fx:"Coach ↑ · BPI ="},
{r:"F4",fv:"Lehigh",dg:"Prairie View A&M",sd:"16v16",open:-1.5,cur:-2.5,tot:142.5,bpi:0.9,sim:5.1,pk:"Lehigh -2.5",c:58,t:"LEAN",e:"Line moved from -1.5 to -2.5. PAT League significantly stronger than SWAC. Yahoo Sports expert Russell also likes Lehigh.",fx:"SOS ↑ · Form ↑"},
{r:"F4",fv:"SMU",dg:"Miami (OH)",sd:"11v11",open:-7.5,cur:-8.5,tot:164.5,bpi:null,sim:5.3,pk:"Miami (OH) +8.5",c:68,t:"BEST BET",e:"Line moved AGAINST us (-7.5 → -8.5) — even more value now. 31-1 team getting 8.5 pts. Miami (OH) 4-0 ATS as dog. SMU only covers 40% as favorite (8-12 ATS). Public is hammering SMU; sharps haven't moved it back.",fx:"Market ↑↑ · Form ↑"},
{r:"E",fv:"Duke",dg:"Siena",sd:"1v16",open:-27.5,cur:-29.5,tot:136.5,bpi:28.9,sim:24.1,pk:"Siena +29.5",c:59,t:"PLAY",e:"Model projects Duke by 24, not 28. BPI says 28.9 — under the current spread. Since 2015, 16-seeds cover 54% when spreads exceed 25. Duke pulls starters at 15-min mark.",fx:"Market ↑↑ · BPI ↑"},
{r:"E",fv:"Ohio State",dg:"TCU",sd:"8v9",open:-2.5,cur:-2.5,tot:149.5,bpi:null,sim:-1.4,pk:"TCU +2.5",c:59,t:"PLAY",e:"Model projects TCU winning outright. TCU beat Florida, won 9 of 11. Better KenPom D (#23). Caesars head trader confirms 'sharper crowd is definitely on TCU.'",fx:"KenPom ↑ · Form ↑ · Sharp ↑"},
{r:"E",fv:"St. John's",dg:"N. Iowa",sd:"5v12",open:-9.5,cur:-9.5,tot:134.5,bpi:null,sim:6.8,pk:"N. Iowa +9.5",c:63,t:"PLAY",e:"12-seeds 8-4 ATS vs 5-seeds since 2022. AI models also side with N. Iowa. Northern Iowa defense KenPom #25. SBD AI model confirms: conf tourney champs are 'battle-tested and playing with house money.'",fx:"History ↑↑ · BPI ↑"},
{r:"E",fv:"Kansas",dg:"Cal Baptist",sd:"4v13",open:-13.5,cur:-14.5,tot:139.5,bpi:null,sim:14.8,pk:"Kansas -14.5",c:55,t:"LEAN",e:"Line moved from -13.5 to -14.5 — public on KU. 4v13 ATS favors chalk (63.5%). Yahoo expert Craig sees this as a 'get-right' spot for Kansas after the Big 12 tourney disaster.",fx:"History ↑ · Sit ↑"},
{r:"E",fv:"Louisville",dg:"S. Florida",sd:"6v11",open:-6.5,cur:-6.5,tot:144.5,bpi:null,sim:-1.2,pk:"S. Florida +6.5",c:72,t:"BEST BET",e:"Mikel Brown Jr. remains OUT. Line holding at -6.5 despite injury — market hasn't fully adjusted. USF 11-game streak, 5 starters avg 11+ PPG. Model projects USF SU win. SportsLine flags Louisville as upset alert.",fx:"Injury ↑↑ · History ↑ · BPI ↑"},
{r:"E",fv:"Mich. State",dg:"N. Dakota St.",sd:"3v14",open:-16.5,cur:-16.5,tot:143.5,bpi:null,sim:19.8,pk:"MSU -16.5",c:62,t:"PLAY",e:"Line steady. Izzo tourney cover rate is 57%. Fears Jr. 20+ in 4 straight. 3v14 ATS favors fav (59%). Multiple model confirmations.",fx:"Coach ↑↑ · Form ↑"},
{r:"E",fv:"UCLA",dg:"UCF",sd:"7v10",open:-5,cur:-6,tot:141.5,bpi:null,sim:-1.1,pk:"UCF +6",c:55,t:"LEAN",e:"AUDIT: Downgraded to LEAN. Sharps laid UCLA -5 (now -6). UCLA's Bilodeau and Dent both expected to play — if healthy, UCLA's 7-1 ATS run (beat Illinois, Nebraska, MSU) is legit. UCF still better by KenPom but this is a true coin flip with conflicting signals.",fx:"Market ↓↓ · KenPom ↑ · Injury ?"},
{r:"E",fv:"UConn",dg:"Furman",sd:"2v15",open:-20.5,cur:-20.5,tot:142.5,bpi:null,sim:16.3,pk:"Furman +20.5",c:58,t:"PLAY",e:"UConn cold: lost to Marquette, routed by SJU in Big East final. 15-seeds cover 52% vs 2-seeds. Recent form drops projected margin below the spread.",fx:"Form ↓↓ · History ↑"},
{r:"S",fv:"Florida",dg:"PV A&M/Lehigh",sd:"1v16",open:-27,cur:-27,tot:138.5,bpi:null,sim:28.6,pk:"Florida -27",c:56,t:"LEAN",e:"Defending champs with best frontcourt in the nation. Model projects 29-pt win. Motivation factor — defending champs don't coast early.",fx:"KenPom ↑ · Sit ↑"},
{r:"S",fv:"Iowa",dg:"Clemson",sd:"9v8",open:-2.5,cur:-2.5,tot:147.5,bpi:null,sim:4.2,pk:"Iowa -2.5",c:61,t:"PLAY",e:"Iowa KenPom 11 spots better. Stirtz (20.2 PPG, 5.1 APG) mismatch. SportsLine also picks a 9-over-8 in South. Multiple cross-model confirmations.",fx:"KenPom ↑ · BPI ↑"},
{r:"S",fv:"Vanderbilt",dg:"McNeese",sd:"5v12",open:-11.5,cur:-12.5,tot:150.5,bpi:9.2,sim:8.4,pk:"McNeese +12.5",c:62,t:"PLAY",e:"Line moved AGAINST us (-11.5 → -12.5) — more value. BPI says Vandy by 9.2 but line is 12.5. 12-seeds 8-4 ATS. McNeese is a three-peat Southland conf tourney champ — poise matters.",fx:"History ↑↑ · BPI ↑"},
{r:"S",fv:"Nebraska",dg:"Troy",sd:"4v13",open:-13.5,cur:-13.5,tot:136.5,bpi:null,sim:14.1,pk:"Nebraska -13.5",c:55,t:"LEAN",e:"KenPom #12 with #7 defense. Huskers desperate for first-ever tourney win. 4v13 ATS favors chalk (63.5%). Troy competed at USC in 3OT though.",fx:"KenPom ↑ · Sit ↑"},
{r:"S",fv:"N. Carolina",dg:"VCU",sd:"6v11",open:-2,cur:-2.5,tot:154.5,bpi:null,sim:1.4,pk:"VCU +2.5",c:64,t:"PLAY",e:"Yahoo expert Russell's featured bet. He notes UNC's power rating dropped post-Wilson injury and that 'the current edition is not as good as the rating.' VCU has lost once since Jan 10, won 6 straight including A-10 title.",fx:"Injury ↑↑ · Expert ↑ · History ↑"},
{r:"S",fv:"Illinois",dg:"Penn",sd:"3v14",open:-21.5,cur:-24.5,tot:148.5,bpi:null,sim:22.8,pk:"Penn +24.5",c:54,t:"LEAN",e:"AUDIT: Downgraded to LEAN. Line inflated from -21.5 to -24.5 by public money — 1.7 pts of dead value. But Illinois has the #1 offense in KenPom and can go nuclear. TJ Power is legit but this is a 3v14 where the 3-seed covers 59% historically. Low conviction — the margin between model and line is thin.",fx:"Market ↑ · KenPom ↓"},
{r:"S",fv:"Saint Mary's",dg:"Texas A&M",sd:"7v10",open:-2.5,cur:-2.5,tot:148.5,bpi:null,sim:1.8,pk:"Texas A&M +2.5",c:57,t:"LEAN",e:"SportsLine picks A&M. SBD AI model's featured pick — 'SEC-caliber physicality in a rock fight.' A&M avg 87.7 PPG (#9) with 3 senior starters. SMC lost WCC semis to Santa Clara — cold form.",fx:"SOS ↑ · BPI ↑"},
{r:"S",fv:"Houston",dg:"Idaho",sd:"2v15",open:-22.5,cur:-23.5,tot:136.5,bpi:25.1,sim:24.6,pk:"Houston -23.5",c:57,t:"LEAN",e:"Line moved from -22.5 to -23.5 — public on Houston. BPI says 25.1. Total is low (136.5) signaling a defensive grind where Houston's elite D covers.",fx:"KenPom ↑ · BPI ↑"},
{r:"W",fv:"Arizona",dg:"LIU",sd:"1v16",open:-29.5,cur:-31.5,tot:145.5,bpi:null,sim:32.4,pk:"Arizona -31.5",c:59,t:"PLAY",e:"Arizona peaking: unbeaten since Feb 14, Big 12 title, 4 top-10 wins in a month. KenPom #3 with elite two-way balance. Model says 32-pt win. Cover even at the inflated line.",fx:"Form ↑↑ · KenPom ↑"},
{r:"W",fv:"Utah State",dg:"Villanova",sd:"9v8",open:-1.5,cur:-3,tot:141.5,bpi:null,sim:2.4,pk:"Utah St. -3",c:60,t:"PLAY",e:"SHARP CONFIRMED: SuperBook risk mgr Crespi called this 'solid early sharp move, from -1.5 to -3.' MW POY Falslev (16.1 PPG, 42% from 3). Utah State 3-0 SU/ATS through MW tourney. Sharps + model + momentum all align.",fx:"Market ↑↑ · Form ↑ · KenPom ="},
{r:"W",fv:"Wisconsin",dg:"High Point",sd:"5v12",open:-12.5,cur:-9.5,tot:150.5,bpi:null,sim:7.6,pk:"High Point +9.5",c:64,t:"PLAY",e:"MASSIVE line movement: opened -12.5, now -9.5 at BetMGM. Sharps hammered High Point so hard the line dropped 3 points. HP on 14-game streak. Model says only 7.6-pt win.",fx:"Market ↑↑↑ · History ↑"},
{r:"W",fv:"Arkansas",dg:"Hawai'i",sd:"4v13",open:-15.5,cur:-15.5,tot:147.5,bpi:null,sim:15.8,pk:"Arkansas -15.5",c:55,t:"LEAN",e:"SEC champs with Acuff Jr. (30 in final). KenPom #6 offense. Cross-country Portland flight is minor concern.",fx:"Form ↑ · Sit ↓"},
{r:"W",fv:"BYU",dg:"11-seed",sd:"6v11",open:null,cur:-4.5,tot:146.5,bpi:null,sim:-1.6,pk:"11-seed +4.5",c:67,t:"BEST BET",e:"BYU declining and one-dimensional beyond Dybantsa. Model projects 11-seed SU win. Every year since 2005, at least one 11 beats a 6. This is the most likely spot.",fx:"Form ↓↓ · History ↑↑"},
{r:"W",fv:"Gonzaga",dg:"Kennesaw St.",sd:"3v14",open:-18.5,cur:-19.5,tot:157.5,bpi:19.2,sim:20.1,pk:"Gonzaga -19.5",c:56,t:"LEAN",e:"Line moved from -18.5 to -19.5. BPI says 19.2 — right on the number. Few hasn't lost before R2 since 2008.",fx:"Coach ↑ · BPI ="},
{r:"W",fv:"Miami (FL)",dg:"Missouri",sd:"7v10",open:-1.5,cur:-1.5,tot:148.5,bpi:null,sim:-0.8,pk:"Missouri +1.5",c:57,t:"PLAY",e:"Model has Mizzou SU win. Game in St. Louis, 2hrs from campus — de facto home game. Situational edge is real.",fx:"Sit ↑↑ · Form ="},
{r:"W",fv:"Purdue",dg:"Queens",sd:"2v15",open:-23.5,cur:-24.5,tot:143.5,bpi:null,sim:26.1,pk:"Purdue -24.5",c:57,t:"PLAY",e:"Big Ten champs. Braden Smith best PG in CBB (9.0 APG). Queens first-time team. Momentum covers.",fx:"Form ↑↑ · KenPom ↑"},
{r:"MW",fv:"Michigan",dg:"UMBC/Howard",sd:"1v16",open:-26,cur:-26,tot:139.5,bpi:null,sim:24.8,pk:"16-seed +26",c:55,t:"LEAN",e:"Model says 25-pt win. Michigan's ceiling lowered without Cason. 16-seeds cover 54% since 2015.",fx:"Injury ↓ · History ↑"},
{r:"MW",fv:"Georgia",dg:"Saint Louis",sd:"8v9",open:-1.5,cur:-2.5,tot:170.5,bpi:null,sim:-0.6,pk:"Saint Louis +2.5",c:60,t:"PLAY",e:"Highest total on board (170.5). SLU top-10 in PPG, FG%, 3P%. Georgia allows 79.2 PPG (worst in 50 yrs). SportsLine also picks upset.",fx:"KenPom = · Form ↑ · BPI ↑"},
{r:"MW",fv:"Texas Tech",dg:"Akron",sd:"5v12",open:-8.5,cur:-7.5,tot:137.5,bpi:null,sim:4.2,pk:"Akron +7.5",c:70,t:"BEST BET",e:"Confirmed sharp move: line dropped from -8.5 to -7.5. TT 0-3 SU/ATS, missing Toppin (ACL). Akron 19-1 in last 20. 12-seeds 8-4 ATS since '22. ALL 8 factors align.",fx:"All factors ↑↑"},
{r:"MW",fv:"Alabama",dg:"Hofstra",sd:"4v13",open:-12.5,cur:-12.5,tot:155.5,bpi:null,sim:11.4,pk:"Hofstra +12.5",c:58,t:"PLAY",e:"Alabama allows 83.5 PPG — worst of any tourney team. Lost SEC tourney opener. Model projects only 11-pt win vs 12.5 spread.",fx:"KenPom ↓↓ · Form ↓"},
{r:"MW",fv:"Tennessee",dg:"SMU/Miami(OH)",sd:"6v11",open:-6,cur:-6,tot:135.5,bpi:null,sim:4.8,pk:"11-seed +6",c:59,t:"PLAY",e:"Tennessee offense is awful (#173 eFG%). 11-seeds cover 56% vs 6-seeds. Every tourney since 2005 has at least one 11-over-6.",fx:"History ↑ · KenPom ↓"},
{r:"MW",fv:"Virginia",dg:"Wright St.",sd:"3v14",open:-17.5,cur:-17.5,tot:126.5,bpi:null,sim:18.4,pk:"Virginia -17.5",c:56,t:"LEAN",e:"Lowest R1 total (126.5). UVA defensive identity grinds opponents. 3v14 ATS favors chalk (59%).",fx:"KenPom ↑ · History ↑"},
{r:"MW",fv:"Kentucky",dg:"Santa Clara",sd:"7v10",open:-2.5,cur:-3.5,tot:160.5,bpi:6.0,sim:2.1,pk:"Santa Clara +3.5",c:62,t:"PLAY",e:"Kentucky 4-6 in final 10, 10-11 ATS as favorite. Santa Clara 26-8, 17-4 since January. Line moved on public name-brand money. Yahoo, NBC Sports pick SC upset. First tourney since 1996 = house money.",fx:"Form ↑↑ · History ↑ · Expert ↑"},
{r:"MW",fv:"Iowa State",dg:"Tennessee St.",sd:"2v15",open:-23.5,cur:-25.5,tot:133.5,bpi:null,sim:27.2,pk:"Iowa St. -25.5",c:58,t:"PLAY",e:"ISU forces ~20 PPG off turnovers. Jefferson at All-American level. Model projects 27-pt win. Wiseguys gravitate toward Iowa State per Caesars trader.",fx:"KenPom ↑ · Form ↑ · Sharp ↑"}
];

const H1=[
{time:"12:15p",tv:"CBS",site:"Greenville",fv:"Ohio State",dg:"TCU",sd:"8v9",fgT:149.5,h1L:70.5,h1P:66.8,pk:"UNDER 70.5",c:62,t:"PLAY",bk:"Both below-avg tempo. TCU defense #23 KenPom. ~32 1H poss × ~2.09 PPP = ~67. Cold-start drops to ~65-67.",eg:"FG total 149.5 qualifies for 67.7% 1H under bucket."},
{time:"2:50p",tv:"CBS",site:"Greenville",fv:"Duke",dg:"Siena",sd:"1v16",fgT:136.5,h1L:64.5,h1P:63.2,pk:"UNDER 64.5",c:57,t:"LEAN",bk:"Siena will slow the game. 1v16 1H scoring averages only 46% of FG total.",eg:"Duke may not press accelerator until 2H."},
{time:"1:30p",tv:"TNT",site:"Buffalo",fv:"Louisville",dg:"S. Florida",sd:"6v11",fgT:144.5,h1L:68.5,h1P:69.4,pk:"OVER 68.5",c:58,t:"PLAY",bk:"ONLY OVER on Thursday. Both top-20 scoring. 60+ combined 3PA/game.",eg:"Two run-and-gun teams + massive 3PA rate = rare 1H Over."},
{time:"4:05p",tv:"TNT",site:"Buffalo",fv:"Mich. State",dg:"N. Dakota St.",sd:"3v14",fgT:143.5,h1L:67.5,h1P:64.1,pk:"UNDER 67.5",c:63,t:"PLAY",bk:"NDSU 65.8 tempo — well below D1 avg. Izzo teams start 1H conservatively.",eg:"Classic tempo mismatch: regresses to slower pace."},
{time:"7:10p",tv:"CBS",site:"Buffalo",fv:"Michigan",dg:"UMBC/Howard",sd:"1v16",fgT:139.5,h1L:65.5,h1P:62.8,pk:"UNDER 65.5",c:64,t:"PLAY",bk:"Michigan #1 defense + below-avg tempo. 16-seed struggles to score 25 in 1H.",eg:"Michigan's elite D + slow tempo = strongest Thursday under signal."},
{time:"9:45p",tv:"CBS",site:"Buffalo",fv:"Georgia",dg:"Saint Louis",sd:"8v9",fgT:170.5,h1L:81.5,h1P:79.8,pk:"UNDER 81.5",c:60,t:"PLAY",bk:"Highest total but 1H under hits 67.7% when FG ≥149. Opening 1H more structured.",eg:"170.5 screams Over — that's the trap."},
{time:"12:40p",tv:"truTV",site:"OKC",fv:"Nebraska",dg:"Troy",sd:"4v13",fgT:136.5,h1L:64.5,h1P:62.2,pk:"UNDER 64.5",c:61,t:"PLAY",bk:"Nebraska #7 defense at 5th-slowest tempo (65.2). Biggest game in program history = nerves.",eg:"Defensive identity + slow tempo + nerves = 1H under."},
{time:"3:15p",tv:"truTV",site:"OKC",fv:"Vanderbilt",dg:"McNeese",sd:"5v12",fgT:150.5,h1L:71.5,h1P:68.4,pk:"UNDER 71.5",c:59,t:"PLAY",bk:"McNeese three-peat Southland champ — disciplined. FG 150.5 qualifies for 67.7% under bucket.",eg:"Small-conf champs play controlled 1H basketball."},
{time:"7:35p",tv:"truTV",site:"OKC",fv:"Houston",dg:"Idaho",sd:"2v15",fgT:136.5,h1L:63.5,h1P:59.8,pk:"UNDER 63.5",c:66,t:"BEST BET",bk:"Houston #6 defense at slow tempo. Idaho AdjOE of 96.4 will be overwhelmed. 1H of ~32-28.",eg:"STRONGEST 1H UNDER. Houston D + Idaho ineptitude = scoring desert."},
{time:"10:10p",tv:"truTV",site:"OKC",fv:"Illinois",dg:"Penn",sd:"3v14",fgT:148.5,h1L:70.5,h1P:69.8,pk:"LEAN UNDER",c:52,t:"SKIP",bk:"Illinois #1 offense at above-avg tempo. Model: 69.8 vs 70.5 — no edge.",eg:"No meaningful edge. Pass."},
{time:"1:50p",tv:"TBS",site:"Portland",fv:"Wisconsin",dg:"High Point",sd:"5v12",fgT:150.5,h1L:71.5,h1P:70.8,pk:"UNDER 71.5",c:55,t:"LEAN",bk:"Wisconsin dictates pace at 68.8. FG 150.5 qualifies for high-total under. Razor thin.",eg:"Marginal edge. Low conviction."},
{time:"4:25p",tv:"TBS",site:"Portland",fv:"Arkansas",dg:"Hawai'i",sd:"4v13",fgT:147.5,h1L:69.5,h1P:66.2,pk:"UNDER 69.5",c:63,t:"PLAY",bk:"MASSIVE tempo mismatch: Arkansas 74.6 vs Hawai'i 64.8 — 10-possession gap.",eg:"Largest tempo gap on Thursday. Hawai'i neutralizes transition."},
{time:"7:25p",tv:"TBS",site:"Portland",fv:"BYU",dg:"11-seed",sd:"6v11",fgT:146.5,h1L:69.5,h1P:66.4,pk:"UNDER 69.5",c:60,t:"PLAY",bk:"11-seed plays conservative. BYU one-dimensional. 6v11 underdogs play patient 1H.",eg:"Upset-prone matchup = patient 1H basketball."},
{time:"10:00p",tv:"TBS",site:"Portland",fv:"Gonzaga",dg:"Kennesaw St.",sd:"3v14",fgT:157.5,h1L:74.5,h1P:72.8,pk:"UNDER 74.5",c:58,t:"PLAY",bk:"Gonzaga missing Huff (17.8 PPG). KSU won't match pace. FG 157.5 = high-total under.",eg:"Huff injury depresses 1H ceiling."},
{time:"9:25p",tv:"TNT",site:"OKC",fv:"Saint Mary's",dg:"Texas A&M",sd:"7v10",fgT:148.5,h1L:70.5,h1P:66.4,pk:"UNDER 70.5",c:64,t:"PLAY",bk:"Saint Mary's SLOWEST tempo Thursday (64.2). They dictate pace. ~30 1H poss × 2.21 PPP = ~66.",eg:"SMC pace control is the biggest 1H under factor Thursday."}
];

const RM={F4:{n:"First Four",c:"#D4A017"},E:{n:"East",c:"#4A90D9"},S:{n:"South",c:"#D94A4A"},W:{n:"West",c:"#3DAA6D"},MW:{n:"Midwest",c:"#8E6CC2"}};

/* ══════════════════════════════════════════════════════════════
   APP COMPONENT
   ══════════════════════════════════════════════════════════════ */

function App(){
  const[tab,setTab]=useState("picks");
  const[reg,setReg]=useState("ALL");
  const[exp,setExp]=useState(null);
  const[h1x,setH1x]=useState(null);
  const fil=reg==="ALL"?G:G.filter(g=>g.r===reg);
  const bests=G.filter(g=>g.t==="BEST BET");
  const tc={"BEST BET":{bg:"#FFF8E7",fg:"#B8860B",bd:"#E8D5A0"},PLAY:{bg:"#EFF8EF",fg:"#2D7A3A",bd:"#C5DEC5"},LEAN:{bg:"#EFF3F8",fg:"#5A7A9A",bd:"#D0DCE8"}};

  const Badge=({t})=>{const s=tc[t]||tc.LEAN;return <span style={{fontFamily:"'DM Sans'",fontSize:10,fontWeight:700,padding:"3px 10px",borderRadius:12,background:s.bg,color:s.fg,border:"1px solid "+s.bd,letterSpacing:.8}}>{t}</span>};
  const Bar=({v})=><div style={{display:"flex",alignItems:"center",justifyContent:"flex-end",gap:6}}><div style={{width:52,height:5,background:"#EDEBE5",borderRadius:3}}><div style={{width:v+"%",height:5,borderRadius:3,background:v>=65?"#B8860B":v>=55?"#2D7A3A":"#5A7A9A"}}/></div><span style={{fontFamily:"'IBM Plex Mono'",fontSize:12,fontWeight:600,color:v>=65?"#B8860B":v>=55?"#2D7A3A":"#5A7A9A"}}>{v}%</span></div>;

  return(
    <div style={{background:"#FAFAF7",color:"#2C2C2C",minHeight:"100vh"}}>
      <header style={{background:"#fff",borderBottom:"1px solid #E8E5DD"}}>
        <div style={{maxWidth:920,margin:"0 auto",padding:"28px 24px 0"}}>
          <div style={{display:"flex",alignItems:"baseline",gap:10,flexWrap:"wrap"}}>
            <h1 style={{fontFamily:"'Source Serif 4',Georgia,serif",fontSize:28,fontWeight:700,color:"#1A1A1A",letterSpacing:-.5}}>March Madness 2026</h1>
            <span style={{fontFamily:"'DM Sans'",fontSize:11,fontWeight:600,color:"#B8860B",letterSpacing:1.5}}>PREDICTION ENGINE v3</span>
          </div>
          <p style={{fontFamily:"'DM Sans'",fontSize:13,color:"#999",margin:"4px 0 16px"}}>8-factor model · Live line tracking · 10K sims/game · Cross-validated with BPI, SportsLine, Silver Bulletin</p>
          <nav style={{display:"flex",marginLeft:-20,overflowX:"auto"}}>
            {[["picks","All Picks"],["best","Best Bets"],["h1","Thu 1H O/U"],["model","Methodology"]].map(([id,l])=>
              <button key={id} className={"nb "+(tab===id?"on":"")} onClick={()=>setTab(id)}>{l}</button>
            )}
          </nav>
        </div>
      </header>

      <main style={{maxWidth:920,margin:"0 auto",padding:"24px 24px 80px"}}>
        <p style={{fontFamily:"'DM Sans'",fontSize:11,color:"#C0392B",marginBottom:16}}>⚠️ For entertainment and analysis only. Please gamble responsibly. 1-800-GAMBLER.</p>

        {/* ALL PICKS */}
        {tab==="picks"&&<>
          <div style={{display:"flex",gap:6,marginBottom:20,flexWrap:"wrap"}}>
            {[["ALL","All Games"],["F4","First Four"],["E","East"],["S","South"],["W","West"],["MW","Midwest"]].map(([k,l])=>
              <button key={k} className={"pl "+(reg===k?"on":"")} onClick={()=>setReg(k)}>{l}</button>
            )}
          </div>
          <div className="stats-grid" style={{display:"grid",gridTemplateColumns:"repeat(4,1fr)",gap:10,marginBottom:24}}>
            {[{n:bests.length,l:"Best Bets",s:"Highest conviction",a:"#B8860B",b:"#FFF8E7"},{n:G.filter(g=>g.t==="PLAY").length,l:"Plays",s:"Strong edge",a:"#2D7A3A",b:"#EFF8EF"},{n:G.filter(g=>g.t==="LEAN").length,l:"Leans",s:"Slight edge",a:"#5A7A9A",b:"#EFF3F8"},{n:G.filter(g=>g.sim<0).length,l:"Dog SU Wins",s:"Underdog projected",a:"#C0392B",b:"#FDF0F0"}].map((s,i)=>
              <div key={i} style={{background:s.b,borderRadius:10,padding:"14px 16px",textAlign:"center",border:"1px solid "+s.a+"15"}}>
                <div style={{fontFamily:"'DM Sans'",fontSize:26,fontWeight:700,color:s.a}}>{s.n}</div>
                <div style={{fontFamily:"'DM Sans'",fontSize:12,fontWeight:600,color:s.a}}>{s.l}</div>
                <div style={{fontFamily:"'DM Sans'",fontSize:10,color:"#aaa",marginTop:2}}>{s.s}</div>
              </div>
            )}
          </div>
          {fil.map((g,i)=>{const o=exp===g.r+"-"+i;const s=tc[g.t]||tc.LEAN;const moved=g.open!==null&&g.open!==g.cur;return(
            <div key={g.r+"-"+i} className="card" onClick={()=>setExp(o?null:g.r+"-"+i)} style={{background:"#fff",borderRadius:10,border:"1px solid #E8E5DD",marginBottom:8,padding:"16px 20px",boxShadow:"0 1px 3px rgba(0,0,0,.03)"}}>
              <div style={{display:"flex",justifyContent:"space-between",alignItems:"flex-start",gap:12,flexWrap:"wrap"}}>
                <div style={{flex:1,minWidth:240}}>
                  <div style={{display:"flex",alignItems:"center",gap:8,marginBottom:6}}>
                    <Badge t={g.t}/><span style={{fontFamily:"'IBM Plex Mono'",fontSize:10,color:(RM[g.r]||{}).c,fontWeight:500}}>{(RM[g.r]||{}).n}</span><span style={{fontFamily:"'IBM Plex Mono'",fontSize:10,color:"#bbb"}}>{g.sd}</span>
                    {moved&&<span style={{fontFamily:"'IBM Plex Mono'",fontSize:9,color:Math.abs(g.cur)>Math.abs(g.open)?"#9B59B6":"#E67E22",fontWeight:600}}>{g.open}→{g.cur}</span>}
                  </div>
                  <div style={{fontSize:17,fontWeight:600,color:"#1A1A1A",marginBottom:4}}>{g.fv} <span style={{color:"#aaa",fontWeight:400,fontSize:14}}>({g.cur})</span><span style={{color:"#ccc",margin:"0 6px",fontSize:12}}>vs</span>{g.dg}</div>
                  <div style={{fontFamily:"'IBM Plex Mono'",display:"flex",gap:14,fontSize:11,color:"#999",flexWrap:"wrap"}}>
                    <span>O/U {g.tot}</span><span style={{color:g.sim<0?"#C0392B":"#2D7A3A",fontWeight:500}}>Sim: {g.sim>0?"+":""}{g.sim}</span>
                    {g.bpi!=null&&<span style={{color:"#4A90D9"}}>BPI: {g.bpi>0?"+":""}{g.bpi}</span>}
                  </div>
                </div>
                <div style={{textAlign:"right",minWidth:130}}>
                  <div style={{fontFamily:"'DM Sans'",fontSize:16,fontWeight:700,color:g.t==="BEST BET"?"#B8860B":"#1A1A1A",marginBottom:6}}>→ {g.pk}</div>
                  <Bar v={g.c}/>
                </div>
              </div>
              {o&&<div style={{marginTop:12,paddingTop:12,borderTop:"1px solid #F0EDE5"}}>
                <p style={{fontSize:14,color:"#555",lineHeight:1.7,marginBottom:8}}>{g.e}</p>
                <div style={{fontFamily:"'IBM Plex Mono'",fontSize:11,color:"#aaa"}}>Key factors: {g.fx}</div>
              </div>}
            </div>
          )})}
        </>}

        {/* BEST BETS */}
        {tab==="best"&&<>
          <h2 style={{fontSize:22,fontWeight:700,marginBottom:6}}>Highest-Conviction Plays</h2>
          <p style={{fontFamily:"'DM Sans'",fontSize:14,color:"#777",lineHeight:1.7,marginBottom:20}}>Games where ≥3 model factors independently align, confirmed by at least one external model or sharp money signal.</p>
          {bests.map((g,i)=>{const moved=g.open!=null&&g.open!==g.cur;return(
            <div key={i} style={{background:"#fff",borderRadius:12,border:"1px solid #E8D5A0",marginBottom:12,padding:"20px 24px",boxShadow:"0 2px 8px rgba(184,134,11,.06)"}}>
              <div style={{display:"flex",justifyContent:"space-between",alignItems:"flex-start",gap:12,flexWrap:"wrap"}}>
                <div style={{flex:1}}>
                  <div style={{display:"flex",alignItems:"center",gap:8,marginBottom:8}}>
                    <Badge t="BEST BET"/><span style={{fontFamily:"'IBM Plex Mono'",fontSize:10,color:(RM[g.r]||{}).c,fontWeight:500}}>{(RM[g.r]||{}).n} · {g.sd}</span>
                    {moved&&<span style={{fontFamily:"'IBM Plex Mono'",fontSize:9,padding:"2px 6px",borderRadius:8,background:"#F3E8FF",color:"#7C3AED",fontWeight:600}}>{g.open}→{g.cur}</span>}
                  </div>
                  <div style={{fontSize:19,fontWeight:600,color:"#1A1A1A",marginBottom:4}}>{g.fv} <span style={{color:"#aaa",fontWeight:400,fontSize:15}}>({g.cur})</span><span style={{color:"#ccc",margin:"0 8px",fontSize:13}}>vs</span>{g.dg}</div>
                  <div style={{fontFamily:"'IBM Plex Mono'",display:"flex",gap:16,fontSize:11,color:"#999"}}><span>O/U {g.tot}</span><span style={{color:g.sim<0?"#C0392B":"#2D7A3A",fontWeight:500}}>Sim: {g.sim>0?"+":""}{g.sim}</span><span>{g.fx}</span></div>
                </div>
                <div style={{textAlign:"right"}}><div style={{fontFamily:"'DM Sans'",fontSize:18,fontWeight:700,color:"#B8860B",marginBottom:6}}>→ {g.pk}</div><Bar v={g.c}/></div>
              </div>
              <p style={{fontSize:14,color:"#555",lineHeight:1.7,marginTop:10,paddingTop:10,borderTop:"1px solid #F5ECD5"}}>{g.e}</p>
            </div>
          )})}
        </>}

        {/* 1H O/U */}
        {tab==="h1"&&<>
          <div style={{background:"#FFF8E7",border:"1px solid #E8D5A0",borderRadius:12,padding:"20px 24px",marginBottom:20}}>
            <h3 style={{fontFamily:"'DM Sans'",fontSize:14,fontWeight:700,color:"#8B6914",marginBottom:8}}>The Edge: Why 1H Unders Work in March</h3>
            <div style={{fontSize:14,color:"#7A6A3A",lineHeight:1.8}}><strong>55%:</strong> 1H unders hit at ~55% since 2015. <strong>67.7%:</strong> When FG total is 149+, 1H unders go 21-10 over last five tourneys. <strong>Why?</strong> First-game jitters, tighter officiating, conservative coaching. Teams score 3-5% less in 1H of first tourney game.</div>
          </div>
          <div className="stats-grid" style={{display:"grid",gridTemplateColumns:"repeat(4,1fr)",gap:10,marginBottom:20}}>
            {[{n:"13",l:"Unders",s:"of 16 games",a:"#2D7A3A",b:"#EFF8EF"},{n:"1",l:"Over",s:"Louisville/USF",a:"#C0392B",b:"#FDF0F0"},{n:"1",l:"Best Bet",s:"HOU/Idaho U63.5",a:"#B8860B",b:"#FFF8E7"},{n:"1",l:"Skip",s:"Illinois/Penn",a:"#999",b:"#F5F5F5"}].map((s,i)=>
              <div key={i} style={{background:s.b,borderRadius:10,padding:"12px 14px",textAlign:"center",border:"1px solid "+s.a+"15"}}>
                <div style={{fontFamily:"'DM Sans'",fontSize:22,fontWeight:700,color:s.a}}>{s.n}</div>
                <div style={{fontFamily:"'DM Sans'",fontSize:11,fontWeight:600,color:s.a}}>{s.l}</div>
                <div style={{fontFamily:"'DM Sans'",fontSize:9,color:"#aaa",marginTop:2}}>{s.s}</div>
              </div>
            )}
          </div>
          {H1.map((g,i)=>{const o=h1x===i;const s=tc[g.t]||tc.LEAN;const isO=g.pk.includes("OVER");return(
            <div key={i} className="card" onClick={()=>setH1x(o?null:i)} style={{background:"#fff",borderRadius:10,border:"1px solid #E8E5DD",marginBottom:8,padding:"16px 20px",boxShadow:"0 1px 3px rgba(0,0,0,.03)"}}>
              <div style={{display:"flex",justifyContent:"space-between",alignItems:"flex-start",gap:12,flexWrap:"wrap"}}>
                <div style={{flex:1,minWidth:240}}>
                  <div style={{display:"flex",alignItems:"center",gap:8,marginBottom:6}}>
                    <Badge t={g.t}/><span style={{fontFamily:"'IBM Plex Mono'",fontSize:10,color:"#999"}}>{g.time} · {g.tv}</span><span style={{fontFamily:"'IBM Plex Mono'",fontSize:10,color:"#bbb"}}>{g.site} · {g.sd}</span>
                  </div>
                  <div style={{fontSize:17,fontWeight:600,color:"#1A1A1A",marginBottom:4}}>{g.fv} <span style={{color:"#ccc",fontSize:12}}>vs</span> {g.dg}</div>
                  <div style={{fontFamily:"'IBM Plex Mono'",display:"flex",gap:14,fontSize:11,color:"#999",flexWrap:"wrap"}}>
                    <span>FG: {g.fgT}</span><span>1H Line: <strong style={{color:"#555"}}>{g.h1L}</strong></span>
                    <span style={{color:g.h1P<g.h1L?"#2D7A3A":"#C0392B"}}>Model: <strong>{g.h1P}</strong></span>
                    <span style={{color:"#bbb"}}>Gap: {(g.h1L-g.h1P).toFixed(1)}</span>
                  </div>
                </div>
                <div style={{textAlign:"right",minWidth:140}}>
                  <div style={{fontFamily:"'DM Sans'",fontSize:15,fontWeight:700,color:g.t==="SKIP"?"#999":isO?"#C0392B":g.t==="BEST BET"?"#B8860B":"#2D7A3A",marginBottom:6}}>→ {g.pk}</div>
                  <Bar v={g.c}/>
                </div>
              </div>
              {o&&<div style={{marginTop:12,paddingTop:12,borderTop:"1px solid #F0EDE5"}}>
                <p style={{fontSize:14,color:"#555",lineHeight:1.7,marginBottom:8}}>{g.bk}</p>
                <div style={{background:"#F9F7F2",borderRadius:8,padding:"10px 14px"}}><div style={{fontFamily:"'DM Sans'",fontSize:11,fontWeight:700,color:"#B8860B",marginBottom:2,letterSpacing:.5}}>EDGE</div><p style={{fontSize:13,color:"#7A6A3A",lineHeight:1.6}}>{g.eg}</p></div>
              </div>}
            </div>
          )})}
        </>}

        {/* METHODOLOGY */}
        {tab==="model"&&<>
          <h2 style={{fontSize:22,fontWeight:700,marginBottom:6}}>Methodology — v3 Tuned</h2>
          <p style={{fontFamily:"'DM Sans'",fontSize:14,color:"#777",lineHeight:1.7,marginBottom:20}}>v3 elevates line movement / sharp money to 15% weight, adds ESPN BPI as a 10% cross-validation factor, and reduces KenPom to 22%. Every pick shows opening→current line movement and BPI projection where available.</p>
          <div style={{background:"#fff",borderRadius:12,border:"1px solid #E8E5DD",padding:24,marginBottom:20}}>
            <h3 style={{fontFamily:"'DM Sans'",fontSize:13,fontWeight:700,marginBottom:16,letterSpacing:.5,textTransform:"uppercase"}}>Factor Weights</h3>
            {MW.map((w,i)=><div key={i} style={{display:"flex",alignItems:"center",gap:12,marginBottom:10}}>
              <div style={{width:130,fontFamily:"'DM Sans'",fontSize:13,fontWeight:600,color:"#444"}}>{w.f}</div>
              <div style={{flex:1,height:10,background:"#F5F3EF",borderRadius:5,overflow:"hidden"}}><div style={{width:w.w*3.5+"%",height:10,borderRadius:5,background:w.c}}/></div>
              <div style={{width:36,fontFamily:"'IBM Plex Mono'",textAlign:"right",fontWeight:600,fontSize:14}}>{w.w}%</div>
            </div>)}
          </div>
          <div style={{background:"#F3E8FF",borderRadius:12,border:"1px solid #D4B8FF",padding:"20px 24px",marginBottom:16}}>
            <h3 style={{fontFamily:"'DM Sans'",fontSize:13,fontWeight:700,color:"#5B21B6",marginBottom:10,letterSpacing:.5,textTransform:"uppercase"}}>v3 Line Movement Corrections</h3>
            <div style={{fontSize:14,color:"#4C1D95",lineHeight:1.8}}><strong>UCLA/UCF:</strong> SuperBook confirmed sharp action on UCLA -5 (now -6). Downgraded UCF to LEAN.<br/><br/><strong>Kentucky/Santa Clara:</strong> Re-corrected to Santa Clara after audit. UK is 4-6 in final 10, 10-11 ATS as fav. Line moved on public money, not sharp.<br/><br/><strong>Utah State/Villanova:</strong> SuperBook Crespi: "solid early sharp move, -1.5 to -3." Upgraded to PLAY.<br/><br/><strong>Wisconsin/High Point:</strong> Line crashed -12.5 to -9.5 — largest sharp-driven R1 move. Confirms HP value.</div>
          </div>
          <div style={{background:"#FDF0F0",borderRadius:12,border:"1px solid #E8C5C5",padding:"20px 24px",marginBottom:16}}>
            <h3 style={{fontFamily:"'DM Sans'",fontSize:13,fontWeight:700,color:"#8B3A3A",marginBottom:10,letterSpacing:.5,textTransform:"uppercase"}}>Defensive Red Flags</h3>
            <div style={{fontSize:14,color:"#6A4040",lineHeight:1.8}}><strong>Alabama (83.5 PPG allowed):</strong> Worst defense of any tournament team. Lost SEC tourney opener.<br/><br/><strong>Georgia (79.2 PPG allowed):</strong> Worst defensive season in 50 years. Five straight tourney losses.</div>
          </div>
          <div style={{background:"#FFF8E7",borderRadius:12,border:"1px solid #E8D5A0",padding:"20px 24px"}}>
            <h3 style={{fontFamily:"'DM Sans'",fontSize:13,fontWeight:700,color:"#8B6914",marginBottom:10,letterSpacing:.5,textTransform:"uppercase"}}>Championship Pick</h3>
            <div style={{fontSize:20,fontWeight:700,color:"#1A1A1A",marginBottom:8}}>Arizona (+400)</div>
            <p style={{fontSize:14,color:"#7A6A3A",lineHeight:1.7}}>KenPom #3 with best two-way balance (#5 offense, #3 defense). Unbeaten since February 14. Won Big 12 title. Nate Silver's model also favors Arizona. CBS experts call Arizona "the safest bet to win the 2026 national championship."</p>
          </div>
        </>}
      </main>
      <footer style={{textAlign:"center",padding:"20px",fontFamily:"'DM Sans'",fontSize:11,color:"#bbb",borderTop:"1px solid #E8E5DD"}}>
        March Madness 2026 Prediction Engine v3 · Last updated: March 16, 2026 · For entertainment only · 1-800-GAMBLER
      </footer>
    </div>
  );
}

ReactDOM.render(<App/>,document.getElementById("root"));
</script>
</body>
</html>
