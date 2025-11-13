# career-buddy-

import 'dart:async';
import 'dart:math';

import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import 'package:google_fonts/google_fonts.dart';


void main() {
  runApp(
    ChangeNotifierProvider(
      create: (_) => PlannerModel(),
      child: CareerBuddyApp(),
    ),
  );
}

class CareerBuddyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final base = ThemeData(
      colorScheme: ColorScheme.fromSeed(seedColor: Colors.indigo),
      useMaterial3: true,
      textTheme: GoogleFonts.interTextTheme(),
    );

    return MaterialApp(
      title: 'CareerBuddy',
      theme: base.copyWith(
        scaffoldBackgroundColor: Colors.grey[50],
      ),
      home: HomeScreen(),
    );
  }
}



class PlannerModel extends ChangeNotifier {
  String name = '';
  int experienceYears = 0;
  List<String> interests = [];
  List<String> skills = [];
  String personality = 'Analytical';

  bool loading = false;
  List<CareerSuggestion> suggestions = [];

  void setName(String v) {
    name = v;
    notifyListeners();
  }

  void setExperience(int v) {
    experienceYears = v;
    notifyListeners();
  }

  void setPersonality(String v) {
    personality = v;
    notifyListeners();
  }

  void toggleInterest(String interest) {
    if (interests.contains(interest)) interests.remove(interest);
    else interests.add(interest);
    notifyListeners();
  }

  void toggleSkill(String skill) {
    if (skills.contains(skill)) skills.remove(skill);
    else skills.add(skill);
    notifyListeners();
  }

  Future<void> generateSuggestions() async {
    loading = true;
    suggestions = [];
    notifyListeners();

    // Simulate network / compute delay with animation-friendly timing.
    await Future.delayed(Duration(milliseconds: 700));

    suggestions = MockAI.generateSuggestions(
      name: name,
      experienceYears: experienceYears,
      interests: interests,
      skills: skills,
      personality: personality,
    );

    loading = false;
    notifyListeners();
  }

  void clear() {
    name = '';
    experienceYears = 0;
    interests = [];
    skills = [];
    personality = 'Analytical';
    suggestions = [];
    notifyListeners();
  }
}

class CareerSuggestion {
  final String title;
  final String short;
  final double score; // 0..1
  final List<String> nextSteps;
  final List<String> learningResources;

  CareerSuggestion({
    required this.title,
    required this.short,
    required this.score,
    required this.nextSteps,
    required this.learningResources,
  });
}

class MockAI {
  static final _careerDB = <CareerSuggestion>[
    CareerSuggestion(
      title: 'Data Scientist',
      short: 'Modeling, analytics, ML pipelines.',
      score: .0,
      nextSteps: [
        'Learn Python (numpy/pandas)',
        'Study statistics & ML (sklearn)',
        'Build 3 projects and portfolio'
      ],
      learningResources: ['Coursera ML', 'fast.ai', 'Kaggle'],
    ),
    CareerSuggestion(
      title: 'Product Manager',
      short: 'Roadmap, stakeholders, go-to-market.',
      score: .0,
      nextSteps: [
        'Understand product lifecycle',
        'Talk to users — conduct interviews',
        'Run a small product experiment'
      ],
      learningResources: ['Reforge', 'PM Crash Course', 'Books: Inspired'],
    ),
    CareerSuggestion(
      title: 'Mobile App Developer',
      short: 'Flutter / native apps and UX.',
      score: .0,
      nextSteps: [
        'Master Dart & Flutter',
        'Ship 2 apps to Play/App Store',
        'Learn state management & testing'
      ],
      learningResources: ['Flutter docs', 'Udemy Flutter Bootcamp'],
    ),
    CareerSuggestion(
      title: 'Cloud Engineer',
      short: 'Infrastructure, CI/CD, reliability.',
      score: .0,
      nextSteps: [
        'Learn AWS/GCP fundamentals',
        'Practice infra as code (Terraform)',
        'Build CI/CD pipelines'
      ],
      learningResources: ['AWS training', 'Cloud Architect path']
    ),
    CareerSuggestion(
      title: 'UX / Product Designer',
      short: 'User research, prototypes, design systems.',
      score: .0,
      nextSteps: [
        'Study UX fundamentals & heuristics',
        'Practice Figma and prototyping',
        'Do usability tests'
      ],
      learningResources: ['NNGroup', 'Figma resources']
    ),
  ];

  // Simple heuristic scoring based on overlap
  static List<CareerSuggestion> generateSuggestions({
    required String name,
    required int experienceYears,
    required List<String> interests,
    required List<String> skills,
    required String personality,
  }) {
    final rnd = Random(name.hashCode ^ experienceYears);
    List<CareerSuggestion> out = [];

    for (var c in _careerDB) {
      double score = 0.1;
      // keywords
      if (c.title.toLowerCase().contains('data') &&
          (skills.contains('Python') || skills.contains('ML'))) score += .4;
      if (c.title.toLowerCase().contains('mobile') &&
          (skills.contains('Flutter') || skills.contains('Dart')))
        score += .45;
      if (c.title.toLowerCase().contains('product') &&
          interests.contains('Building')) score += .35;
      if (c.title.toLowerCase().contains('cloud') &&
          skills.contains('DevOps')) score += .4;
      if (c.title.toLowerCase().contains('ux') &&
          interests.contains('Design')) score += .45;

      // personality bias
      if (personality == 'Analytical' &&
          (c.title.contains('Data') || c.title.contains('Cloud')))
        score += .08;
      if (personality == 'Creative' && c.title.contains('Designer')) score += .1;

      // experience adjust
      score += (experienceYears.clamp(0, 5)) * 0.03;

      // interest overlap
      for (var it in interests) {
        if (c.short.toLowerCase().contains(it.toLowerCase())) score += .05;
      }

      // small randomness so results don't feel identical
      score += (rnd.nextDouble() * 0.08);

      // clamp
      score = score.clamp(0.0, 0.99);

      out.add(CareerSuggestion(
        title: c.title,
        short: c.short,
        score: double.parse(score.toStringAsFixed(2)),
        nextSteps: c.nextSteps,
        learningResources: c.learningResources,
      ));
    }

    out.sort((a, b) => b.score.compareTo(a.score));
    return out;
  }
}

/* -------------------------
   UI: Home & Navigation
   ------------------------- */

class HomeScreen extends StatelessWidget {
  @override
  Widget build(BuildContext c) {
    final theme = Theme.of(c);
    return Scaffold(
      body: SafeArea(
        child: Column(
          children: [
            Hero(
              tag: 'app-logo',
              child: _TopCard(),
            ),
            Expanded(
              child: _HomeBody(),
            )
          ],
        ),
      ),
      floatingActionButton: _FloatingMenu(),
    );
  }
}

class _TopCard extends StatelessWidget {
  @override
  Widget build(BuildContext c) {
    return Container(
      margin: EdgeInsets.all(16),
      padding: EdgeInsets.symmetric(vertical: 22, horizontal: 18),
      decoration: BoxDecoration(
        gradient: LinearGradient(
          colors: [Colors.indigo.shade500, Colors.indigo.shade300],
        ),
        borderRadius: BorderRadius.circular(16),
        boxShadow: [BoxShadow(color: Colors.indigo.withOpacity(.15), blurRadius: 10, offset: Offset(0,6))],
      ),
      child: Row(
        children: [
          CircleAvatar(
            radius: 34,
            backgroundColor: Colors.white24,
            child: Icon(Icons.computer, size: 34, color: Colors.white),
          ),
          SizedBox(width: 16),
          Expanded(
            child: AnimatedSwitcher(
              duration: Duration(milliseconds: 450),
              child: Column(
                key: ValueKey<int>(1),
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  Text('CareerBuddy', style: TextStyle(color: Colors.white, fontSize: 20, fontWeight: FontWeight.bold)),
                  SizedBox(height: 6),
                  Text('AI-powered career planning', style: TextStyle(color: Colors.white70)),
                ],
              ),
            ),
          ),
          ElevatedButton.icon(
            onPressed: () => Navigator.of(c).push(_fadeRoute(ProfileScreen())),
            icon: Icon(Icons.person_outline),
            label: Text('Profile'),
            style: ElevatedButton.styleFrom(
              backgroundColor: Colors.white24,
              elevation: 0,
            ),
          )
        ],
      ),
    );
  }
}

class _HomeBody extends StatelessWidget {
  @override
  Widget build(BuildContext c) {
    final model = contextWatch<PlannerModel>(c);
    return SingleChildScrollView(
      padding: EdgeInsets.symmetric(horizontal: 16, vertical: 6),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          Text('Plan your next move', style: TextStyle(fontSize: 20, fontWeight: FontWeight.w700)),
          SizedBox(height: 12),
          _FeatureCard(
            title: 'AI Career Planner',
            subtitle: 'Tell us about your interests and get a tailored path.',
            icon: Icons.auto_stories,
            color: Colors.orange,
            onTap: () => Navigator.of(c).push(_slideFromRight(PlannerScreen())),
          ),
          _FeatureCard(
            title: 'Saved Suggestions',
            subtitle: 'View your last generated plans.',
            icon: Icons.bookmark_outline,
            color: Colors.teal,
            onTap: () {
              if (model.suggestions.isEmpty) {
                ScaffoldMessenger.of(c).showSnackBar(SnackBar(content: Text('No saved suggestions — run planner first.')));
                return;
              }
              Navigator.of(c).push(_slideFromRight(SuggestionsScreen()));
            },
          ),
          _FeatureCard(
            title: 'Resources & Courses',
            subtitle: 'Curated learning resources.',
            icon: Icons.school_outlined,
            color: Colors.purple,
            onTap: () => Navigator.of(c).push(_slideFromRight(ResourcesScreen())),
          ),
          SizedBox(height: 16),
          Text('Quick tips', style: TextStyle(fontSize: 16, fontWeight: FontWeight.w600)),
          SizedBox(height: 8),
          _TipsList(),
        ],
      ),
    );
  }
}

class _FeatureCard extends StatelessWidget {
  final String title, subtitle;
  final IconData icon;
  final Color color;
  final VoidCallback onTap;
  _FeatureCard({required this.title, required this.subtitle, required this.icon, required this.color, required this.onTap});

  @override
  Widget build(BuildContext c) {
    return GestureDetector(
      onTap: onTap,
      child: AnimatedContainer(
        duration: Duration(milliseconds: 350),
        margin: EdgeInsets.symmetric(vertical: 8),
        padding: EdgeInsets.all(12),
        decoration: BoxDecoration(
          color: Colors.white,
          borderRadius: BorderRadius.circular(12),
          boxShadow: [BoxShadow(color: Colors.black12, blurRadius: 8, offset: Offset(0,6))],
        ),
        child: Row(
          children: [
            CircleAvatar(backgroundColor: color.withOpacity(.15), child: Icon(icon, color: color)),
            SizedBox(width: 12),
            Expanded(child: Column(crossAxisAlignment: CrossAxisAlignment.start, children: [
              Text(title, style: TextStyle(fontWeight: FontWeight.w700)),
              SizedBox(height: 4),
              Text(subtitle, style: TextStyle(color: Colors.black54, fontSize: 13)),
            ])),
            Icon(Icons.chevron_right),
          ],
        ),
      ),
    );
  }
}

class _TipsList extends StatelessWidget {
  final tips = [
    'Ship small projects; employers value outcome.',
    'Document learning — write blog posts or case studies.',
    'Talk to professionals over LinkedIn for quick advice.',
  ];
  @override
  Widget build(BuildContext c) {
    return Column(
      children: tips.map((t) => ListTile(leading: Icon(Icons.lightbulb_outline), title: Text(t))).toList(),
    );
  }
}

/* -------------------------
   Planner: Multi-step input
   ------------------------- */

class PlannerScreen extends StatefulWidget {
  @override
  _PlannerScreenState createState() => _PlannerScreenState();
}

class _PlannerScreenState extends State<PlannerScreen> with TickerProviderStateMixin {
  int _step = 0;
  late PageController _pc;
  final _nameCtrl = TextEditingController();
  final _expCtrl = TextEditingController();

  @override
  void initState() {
    _pc = PageController();
    super.initState();
  }

  @override
  void dispose() {
    _pc.dispose();
    _nameCtrl.dispose();
    _expCtrl.dispose();
    super.dispose();
  }

  void _next() {
    if (_step < 2) {
      setState(() => _step += 1);
      _pc.nextPage(duration: Duration(milliseconds: 400), curve: Curves.easeInOut);
    } else {
      // generate suggestions
      final model = Provider.of<PlannerModel>(context, listen: false);
      model.generateSuggestions().then((_) {
        Navigator.of(context).pushReplacement(_slideFromRight(SuggestionsScreen()));
      });
    }
  }

  void _prev() {
    if (_step > 0) {
      setState(() => _step -= 1);
      _pc.previousPage(duration: Duration(milliseconds: 400), curve: Curves.easeInOut);
    } else {
      Navigator.of(context).pop();
    }
  }

  @override
  Widget build(BuildContext c) {
    final model = Provider.of<PlannerModel>(c);
    return Scaffold(
      appBar: AppBar(
        title: Text('Career Planner'),
        elevation: 0,
        backgroundColor: Colors.transparent,
        foregroundColor: Colors.black87,
      ),
      body: Column(
        children: [
          LinearProgressIndicator(value: (_step + 1) / 3, minHeight: 6),
          Expanded(
            child: PageView(
              controller: _pc,
              physics: NeverScrollableScrollPhysics(),
              children: [
                _step1Personal(model),
                _step2SkillsInterests(model),
                _step3Personality(model),
              ],
            ),
          ),
          _bottomBar(model),
        ],
      ),
    );
  }

  Widget _step1Personal(PlannerModel model) {
    _nameCtrl.text = model.name;
    _expCtrl.text = model.experienceYears.toString();
    return Padding(
      padding: EdgeInsets.all(18),
      child: Column(crossAxisAlignment: CrossAxisAlignment.start, children: [
        SizedBox(height: 8),
        Text('Tell us about yourself', style: TextStyle(fontSize: 18, fontWeight: FontWeight.w700)),
        SizedBox(height: 12),
        TextField(
          controller: _nameCtrl,
          decoration: InputDecoration(labelText: 'Name', border: OutlineInputBorder()),
          onChanged: (v) => model.setName(v),
        ),
        SizedBox(height: 12),
        TextField(
          controller: _expCtrl,
          keyboardType: TextInputType.number,
          decoration: InputDecoration(labelText: 'Years of experience', border: OutlineInputBorder()),
          onChanged: (v) => model.setExperience(int.tryParse(v) ?? 0),
        ),
      ]),
    );
  }

  Widget _step2SkillsInterests(PlannerModel model) {
    final interests = ['Building', 'Design', 'Research', 'Teaching', 'Data'];
    final skills = ['Dart', 'Flutter', 'Python', 'ML', 'DevOps', 'React'];

    Widget chipMaker(List<String> list, List<String> selected, Function(String) toggle) {
      return Wrap(
        spacing: 8,
        children: list.map((s) {
          final sel = selected.contains(s);
          return FilterChip(
            label: Text(s),
            selected: sel,
            onSelected: (_) => toggle(s),
          );
        }).toList(),
      );
    }

    return Padding(
      padding: EdgeInsets.all(18),
      child: Column(crossAxisAlignment: CrossAxisAlignment.start, children: [
        Text('Interests', style: TextStyle(fontSize: 18, fontWeight: FontWeight.w700)),
        SizedBox(height: 8),
        chipMaker(interests, model.interests, model.toggleInterest),
        SizedBox(height: 18),
        Text('Skills', style: TextStyle(fontSize: 18, fontWeight: FontWeight.w700)),
        SizedBox(height: 8),
        chipMaker(skills, model.skills, model.toggleSkill),
      ]),
    );
  }

  Widget _step3Personality(PlannerModel model) {
    final personalities = ['Analytical', 'Creative', 'Organized', 'Communicator'];
    return Padding(
      padding: EdgeInsets.all(18),
      child: Column(crossAxisAlignment: CrossAxisAlignment.start, children: [
        Text('Personality', style: TextStyle(fontSize: 18, fontWeight: FontWeight.w700)),
        SizedBox(height: 12),
        ...personalities.map((p) => RadioListTile<String>(
          title: Text(p),
          value: p,
          groupValue: model.personality,
          onChanged: (v) => model.setPersonality(v!),
        ))
      ]),
    );
  }

  Widget _bottomBar(PlannerModel model) {
    return Container(
      padding: EdgeInsets.all(12),
      child: Row(
        children: [
          TextButton.icon(onPressed: _prev, icon: Icon(Icons.chevron_left), label: Text(_step == 0 ? 'Back' : 'Previous')),
          Spacer(),
          ElevatedButton(
            onPressed: model.loading ? null : _next,
            child: Padding(
              padding: EdgeInsets.symmetric(vertical: 12, horizontal: 16),
              child: model.loading ? SizedBox(width: 18, height: 18, child: CircularProgressIndicator(strokeWidth: 2, color: Colors.white)) : Text(_step == 2 ? 'Get Suggestions' : 'Next'),
            ),
          )
        ],
      ),
    );
  }
}

/* -------------------------
   Suggestions Screen (animated)
   ------------------------- */

class SuggestionsScreen extends StatefulWidget {
  @override
  _SuggestionsScreenState createState() => _SuggestionsScreenState();
}

class _SuggestionsScreenState extends State<SuggestionsScreen> {
  final GlobalKey<AnimatedListState> _listKey = GlobalKey();

  @override
  void initState() {
    super.initState();
    // animated entrance of suggestions
    WidgetsBinding.instance.addPostFrameCallback((_) {
      final model = Provider.of<PlannerModel>(context, listen: false);
      if (_listKey.currentState == null) return;
      for (int i = 0; i < model.suggestions.length; i++) {
        Future.delayed(Duration(milliseconds: 120 * i), () {
          _listKey.currentState?.insertItem(i);
        });
      }
    });
  }

  @override
  Widget build(BuildContext c) {
    final model = Provider.of<PlannerModel>(c);
    return Scaffold(
      appBar: AppBar(
        title: Text('Your Suggestions'),
        backgroundColor: Colors.transparent,
        foregroundColor: Colors.black87,
      ),
      body: model.suggestions.isEmpty ? _emptyState(c) : Padding(
        padding: EdgeInsets.all(12),
        child: AnimatedList(
          key: _listKey,
          initialItemCount: model.suggestions.length,
          itemBuilder: (context, index, animation) {
            final s = model.suggestions[index];
            return SizeTransition(
              sizeFactor: animation.drive(CurveTween(curve: Curves.easeOut)),
              child: _SuggestionCard(suggestion: s, index: index),
            );
          },
        ),
      ),
      floatingActionButton: FloatingActionButton.extended(
        icon: Icon(Icons.refresh),
        label: Text('Re-run'),
        onPressed: () {
          model.generateSuggestions();
          Navigator.of(context).pop();
        },
      ),
    );
  }

  Widget _emptyState(BuildContext c) {
    return Center(
      child: Column(mainAxisSize: MainAxisSize.min, children: [
        Icon(Icons.hourglass_empty, size: 64, color: Colors.black26),
        SizedBox(height: 12),
        Text('No suggestions yet', style: TextStyle(fontSize: 16)),
        SizedBox(height: 8),
        ElevatedButton(onPressed: () => Navigator.of(c).push(_slideFromRight(PlannerScreen())), child: Text('Run Planner')),
      ]),
    );
  }
}

class _SuggestionCard extends StatelessWidget {
  final CareerSuggestion suggestion;
  final int index;
  _SuggestionCard({required this.suggestion, required this.index});

  @override
  Widget build(BuildContext c) {
    final color = Colors.primaries[index % Colors.primaries.length];
    return GestureDetector(
      onTap: () => Navigator.of(c).push(_slideFromRight(CareerDetailScreen(suggestion: suggestion))),
      child: Container(
        margin: EdgeInsets.only(bottom: 12),
        padding: EdgeInsets.all(14),
        decoration: BoxDecoration(
          color: Colors.white,
          borderRadius: BorderRadius.circular(12),
          border: Border.all(color: Colors.grey.withOpacity(.08)),
          boxShadow: [BoxShadow(color: Colors.black12, blurRadius: 8, offset: Offset(0,6))],
        ),
        child: Row(children: [
          Hero(
            tag: 'career-${suggestion.title}',
            child: CircleAvatar(backgroundColor: color.shade100, child: Icon(Icons.star, color: color.shade700)),
          ),
          SizedBox(width: 12),
          Expanded(child: Column(crossAxisAlignment: CrossAxisAlignment.start, children: [
            Row(children: [
              Expanded(child: Text(suggestion.title, style: TextStyle(fontWeight: FontWeight.w800, fontSize: 16))),
              Container(
                padding: EdgeInsets.symmetric(horizontal: 8, vertical: 6),
                decoration: BoxDecoration(color: Colors.grey[100], borderRadius: BorderRadius.circular(10)),
                child: Text('${(suggestion.score*100).round()}%', style: TextStyle(fontWeight: FontWeight.w700)),
              )
            ]),
            SizedBox(height: 6),
            Text(suggestion.short, style: TextStyle(color: Colors.black54)),
          ]))
        ]),
      ),
    );
  }
}

/* -------------------------
   Career detail
   ------------------------- */

class CareerDetailScreen extends StatelessWidget {
  final CareerSuggestion suggestion;
  CareerDetailScreen({required this.suggestion});

  @override
  Widget build(BuildContext c) {
    return Scaffold(
      appBar: AppBar(backgroundColor: Colors.transparent, foregroundColor: Colors.black87, title: Text(suggestion.title)),
      body: Padding(
        padding: EdgeInsets.all(16),
        child: Column(crossAxisAlignment: CrossAxisAlignment.start, children: [
          Hero(tag: 'career-${suggestion.title}', child: CircleAvatar(radius: 36, child: Icon(Icons.star_outline, size: 36))),
          SizedBox(height: 12),
          Text(suggestion.short, style: TextStyle(fontSize: 16)),
          SizedBox(height: 16),
          Text('Next steps', style: TextStyle(fontWeight: FontWeight.w700)),
          SizedBox(height: 8),
          ...suggestion.nextSteps.map((s) => ListTile(leading: Icon(Icons.check_circle_outline), title: Text(s))).toList(),
          SizedBox(height: 8),
          Text('Resources', style: TextStyle(fontWeight: FontWeight.w700)),
          ...suggestion.learningResources.map((r) => ListTile(leading: Icon(Icons.link), title: Text(r))).toList(),
        ]),
      ),
    );
  }
}

/* -------------------------
   Resources & Profile & Settings
   ------------------------- */

class ResourcesScreen extends StatelessWidget {
  @override
  Widget build(BuildContext c) {
    final resources = [
      {'title': 'Coursera - ML', 'desc': 'Intro to ML'},
      {'title': 'fast.ai', 'desc': 'Practical deep learning'},
      {'title': 'Flutter docs', 'desc': 'Official Flutter tutorials'},
    ];
    return Scaffold(
      appBar: AppBar(title: Text('Resources')),
      body: ListView.separated(
        itemBuilder: (ctx, i) => ListTile(
          leading: Icon(Icons.school_outlined),
          title: Text(resources[i]['title']!),
          subtitle: Text(resources[i]['desc']!),
        ),
        separatorBuilder: (_, __) => Divider(),
        itemCount: resources.length,
      ),
    );
  }
}

class ProfileScreen extends StatelessWidget {
  @override
  Widget build(BuildContext c) {
    final model = Provider.of<PlannerModel>(c);
    return Scaffold(
      appBar: AppBar(title: Text('Profile')),
      body: Padding(
        padding: EdgeInsets.all(18),
        child: Column(crossAxisAlignment: CrossAxisAlignment.start, children: [
          Text('Name', style: TextStyle(fontWeight: FontWeight.w700)),
          SizedBox(height: 6),
          Text(model.name.isEmpty ? '—' : model.name),
          SizedBox(height: 12),
          Text('Experience', style: TextStyle(fontWeight: FontWeight.w700)),
          SizedBox(height: 6),
          Text('${model.experienceYears} years'),
          SizedBox(height: 12),
          Text('Interests', style: TextStyle(fontWeight: FontWeight.w700)),
          Wrap(spacing: 8, children: model.interests.map((s) => Chip(label: Text(s))).toList()),
          SizedBox(height: 12),
          Text('Skills', style: TextStyle(fontWeight: FontWeight.w700)),
          Wrap(spacing: 8, children: model.skills.map((s) => Chip(label: Text(s))).toList()),
          Spacer(),
          ElevatedButton.icon(
            onPressed: () {
              model.clear();
              ScaffoldMessenger.of(c).showSnackBar(SnackBar(content: Text('Profile cleared')));
            },
            icon: Icon(Icons.delete_outline),
            label: Text('Clear profile'),
            style: ElevatedButton.styleFrom(backgroundColor: Colors.redAccent),
          )
        ]),
      ),
    );
  }
}

/* -------------------------
   Utilities & routes
   ------------------------- */

Route _slideFromRight(Widget page) {
  return PageRouteBuilder(
    transitionDuration: Duration(milliseconds: 420),
    pageBuilder: (_, a, __) => page,
    transitionsBuilder: (_, a, __, child) {
      final tween = Tween(begin: Offset(1, 0), end: Offset.zero).chain(CurveTween(curve: Curves.easeOutCubic));
      return SlideTransition(position: a.drive(tween), child: FadeTransition(opacity: a, child: child));
    },
  );
}

Route _fadeRoute(Widget page) {
  return PageRouteBuilder(
    transitionDuration: Duration(milliseconds: 360),
    pageBuilder: (_, a, __) => page,
    transitionsBuilder: (_, a, __, child) => FadeTransition(opacity: a, child: child),
  );
}

T contextWatch<T>(BuildContext c) => Provider.of<T>(c);

class _FloatingMenu extends StatefulWidget {
  @override
  __FloatingMenuState createState() => __FloatingMenuState();
}
class __FloatingMenuState extends State<_FloatingMenu> with SingleTickerProviderStateMixin {
  bool open = false;
  late AnimationController _ctrl;
  @override
  void initState() {
    _ctrl = AnimationController(vsync: this, duration: Duration(milliseconds: 350));
    super.initState();
  }
  @override
  void dispose() { _ctrl.dispose(); super.dispose(); }

  void toggle() {
    setState(() => open = !open);
    if (open) _ctrl.forward(); else _ctrl.reverse();
  }

  @override
  Widget build(BuildContext c) {
    return Column(
      mainAxisSize: MainAxisSize.min,
      children: [
        ScaleTransition(
          scale: Tween<double>(begin: 0, end: 1).animate(CurvedAnimation(parent: _ctrl, curve: Interval(0.0, 0.6, curve: Curves.easeOut))),
          child: FloatingActionButton.small(
            heroTag: 'fab1',
            onPressed: () => Navigator.of(c).push(_slideFromRight(ResourcesScreen())),
            child: Icon(Icons.school),
            tooltip: 'Resources',
          ),
        ),
        SizedBox(height: 8),
        ScaleTransition(
          scale: Tween<double>(begin: 0, end: 1).animate(CurvedAnimation(parent: _ctrl, curve: Interval(0.1, 0.7, curve: Curves.easeOut))),
          child: FloatingActionButton.small(
            heroTag: 'fab2',
            onPressed: () => Navigator.of(c).push(_slideFromRight(PlannerScreen())),
            child: Icon(Icons.auto_stories),
            tooltip: 'Planner',
          ),
        ),
        SizedBox(height: 8),
        FloatingActionButton(
          heroTag: 'fab-main',
          onPressed: toggle,
          child: AnimatedIcon(icon: AnimatedIcons.menu_close, progress: _ctrl),
        )
      ],
    );
  }
}
