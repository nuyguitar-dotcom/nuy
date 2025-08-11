import 'package:flutter/material.dart';
import 'package:intl/intl.dart';
import 'package:sqflite/sqflite.dart';
import 'package:path/path.dart' as p;

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await DB.instance.open();
  runApp(const App());
}

class App extends StatelessWidget {
  const App({super.key});
  @override
  Widget build(BuildContext context) => MaterialApp(
        title: 'รวมเลข',
        debugShowCheckedModeBanner: false,
        theme: ThemeData(
          colorScheme: ColorScheme.fromSeed(seedColor: const Color(0xFF6EB5FF)),
          useMaterial3: true,
        ),
        home: const Home(),
      );
}

class Home extends StatefulWidget {
  const Home({super.key});
  @override
  State<Home> createState() => _HomeState();
}

class _HomeState extends State<Home> {
  int idx = 0;
  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('รวมเลข'), centerTitle: true),
        body: [const AddPage(), const SummaryPage(true), const SummaryPage(false)][idx],
        bottomNavigationBar: NavigationBar(
          selectedIndex: idx,
          onDestinationSelected: (i) => setState(() => idx = i),
          destinations: const [
            NavigationDestination(icon: Icon(Icons.add), label: 'ป้อนข้อมูล'),
            NavigationDestination(icon: Icon(Icons.grid_view), label: 'สรุป 2 หลัก'),
            NavigationDestination(icon: Icon(Icons.dataset), label: 'สรุป 3 หลัก'),
          ],
        ),
      );
}

class AddPage extends StatefulWidget {
  const AddPage({super.key});
  @override
  State<AddPage> createState() => _AddPageState();
}

class _AddPageState extends State<AddPage> {
  final number = TextEditingController();
  final amount = TextEditingController();
  @override
  void dispose() { number.dispose(); amount.dispose(); super.dispose(); }

  Future<void> save() async {
    final n = number.text.trim();
    final a = amount.text.trim();
    if (!RegExp(r'^\d{2,3}$').hasMatch(n)) { snack('เลขต้องเป็นตัวเลข 2 หรือ 3 หลัก'); return; }
    if (!RegExp(r'^\d+$').hasMatch(a)) { snack('จำนวนต้องเป็นตัวเลข'); return; }
    final isTwo = n.length == 2;
    final cat = isTwo ? 'TWO' : 'THREE';
    final now = DateTime.now();
    final roundKey = computeRound(now);
    await DB.instance.insert({
      'number': n,
      'amount': int.parse(a),
      'date_text': DateFormat('dd/MM/yyyy').format(now),
      'category': cat,
      'round_key': roundKey,
      'created_at': now.millisecondsSinceEpoch,
    });
    number.clear(); amount.clear();
    snack('บันทึกเรียบร้อย');
  }

  String computeRound(DateTime d) {
    final day = d.day;
    if (day >= 2 && day <= 16) return DateFormat('yyyyMM').format(d) + '_A';
    if (day == 1) { final prev = DateTime(d.year, d.month - 1, 17); return DateFormat('yyyyMM').format(prev) + '_B'; }
    return DateFormat('yyyyMM').format(d) + '_B';
  }

  void snack(String m) => ScaffoldMessenger.of(context).showSnackBar(SnackBar(content: Text(m)));

  @override
  Widget build(BuildContext context) => Padding(
        padding: const EdgeInsets.all(16),
        child: Column(crossAxisAlignment: CrossAxisAlignment.start, children: [
          const Text('ป้อนเลข (2 หรือ 3 หลัก)', style: TextStyle(fontWeight: FontWeight.bold)),
          const SizedBox(height: 6),
          TextField(
            controller: number, keyboardType: TextInputType.number, maxLength: 3,
            decoration: const InputDecoration(border: OutlineInputBorder(), hintText: 'เช่น 05 หรือ 005'),
            onChanged: (v) {
              final d = v.replaceAll(RegExp(r'\D'), '');
              final clip = d.length > 3 ? d.substring(0, 3) : d;
              if (clip != v) number.value = TextEditingValue(text: clip, selection: TextSelection.collapsed(offset: clip.length));
            },
          ),
          const SizedBox(height: 8),
          const Text('จำนวน', style: TextStyle(fontWeight: FontWeight.bold)),
          const SizedBox(height: 6),
          TextField(
            controller: amount, keyboardType: TextInputType.number,
            decoration: const InputDecoration(border: OutlineInputBorder(), hintText: 'เช่น 200'),
            onChanged: (v) {
              final d = v.replaceAll(RegExp(r'\D'), '');
              if (d != v) amount.value = TextEditingValue(text: d, selection: TextSelection.collapsed(offset: d.length));
            },
          ),
          const SizedBox(height: 16),
          SizedBox(width: double.infinity, child: FilledButton.icon(onPressed: save, icon: const Icon(Icons.save), label: const Text('บันทึก'))),
          const SizedBox(height: 10),
          const Text('ระบบจะแยก 2/3 หลัก และรอบวันที่ (2–16, 17–1) ให้อัตโนมัติ', style: TextStyle(fontSize: 12)),
        ]),
      );
}

class SummaryPage extends StatefulWidget {
  final bool two;
  const SummaryPage(this.two, {super.key});
  @override
  State<SummaryPage> createState() => _SummaryPageState();
}

class _SummaryPageState extends State<SummaryPage> {
  String keyRound = '';
  List<Map<String, Object?>> items = [];
  @override
  void initState() { super.initState(); keyRound = _currentRound(); _load(); }

  String _currentRound() {
    final n = DateTime.now(); final d = n.day;
    if (d >= 2 && d <= 16) return DateFormat('yyyyMM').format(n) + '_A';
    if (d == 1) { final prev = DateTime(n.year, n.month - 1, 17); return DateFormat('yyyyMM').format(prev) + '_B'; }
    return DateFormat('yyyyMM').format(n) + '_B';
  }

  Future<void> _load() async {
    final cat = widget.two ? 'TWO' : 'THREE';
    items = await DB.instance.summary(cat, keyRound);
    setState((){});
  }

  Future<void> _shift(bool next) async {
    final sp = keyRound.split('_'); final ym = sp[0]; final ab = sp[1];
    final y = int.parse(ym.substring(0,4)); final m = int.parse(ym.substring(4,6));
    if (next) {
      keyRound = (ab == 'A') ? '${ym}_B' : '${DateFormat('yyyyMM').format(DateTime(y, m+1, 1))}_A';
    } else {
      keyRound = (ab == 'B') ? '${ym}_A' : '${DateFormat('yyyyMM').format(DateTime(y, m-1, 1))}_B';
    }
    await _load();
  }

  @override
  Widget build(BuildContext context) => Column(children: [
    Padding(
      padding: const EdgeInsets.fromLTRB(12,12,12,0),
      child: Row(children: [
        IconButton(onPressed: () => _shift(false), icon: const Icon(Icons.chevron_left)),
        const Expanded(child: Center(child: Text('สรุปเลข', style: TextStyle(fontWeight: FontWeight.bold)))),
        IconButton(onPressed: () => _shift(true), icon: const Icon(Icons.chevron_right)),
      ]),
    ),
    Expanded(child: items.isEmpty
      ? const Center(child: Text('ยังไม่มีข้อมูลในรอบนี้'))
      : ListView.separated(
        itemCount: items.length, separatorBuilder: (_, __) => const Divider(height: 1),
        itemBuilder: (_, i) {
          final it = items[i];
          return ListTile(
            dense: true,
            title: Text(it['number'] as String),
            trailing: Text('${it['total']}', style: const TextStyle(fontWeight: FontWeight.bold)),
            subtitle: (it['last_date'] as String?) != null ? Text('อัปเดตล่าสุด: ${it['last_date']}') : null,
          );
        })),
  ]);
}

class DB {
  DB._(); static final instance = DB._();
  Database? _db;
  Future<void> open() async {
    final path = p.join(await getDatabasesPath(), 'ruamlek.db');
    _db = await openDatabase(path, version: 1, onCreate: (db, v) async {
      await db.execute('''CREATE TABLE entries(
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        number TEXT NOT NULL,
        amount INTEGER NOT NULL,
        date_text TEXT NOT NULL,
        category TEXT NOT NULL,
        round_key TEXT NOT NULL,
        created_at INTEGER NOT NULL
      );''');
      await db.execute('CREATE INDEX idx1 ON entries(category, round_key);');
      await db.execute('CREATE INDEX idx2 ON entries(number);');
    });
  }
  Future<int> insert(Map<String,Object?> data) async => await _db!.insert('entries', data);
  Future<List<Map<String,Object?>>> summary(String category, String roundKey) async {
    return await _db!.rawQuery('''
      SELECT number, SUM(amount) AS total, MAX(date_text) AS last_date
      FROM entries
      WHERE category = ? AND round_key = ?
      GROUP BY number
      ORDER BY total DESC, number ASC
    ''', [category, roundKey]);
  }
}
