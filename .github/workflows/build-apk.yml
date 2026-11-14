import 'package:flutter/material.dart';
import 'package:supabase_flutter/supabase_flutter.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  
  // HUMANSGRAM BACKEND - Seniki
  await Supabase.initialize(
    url: 'https://hpuqkuuaecaqqndvhbyx.supabase.co',
    anonKey: 'sb_publishable_k2szgtNGN-Osv9bMXyKccw_BATsiRwU',
  );
  
  runApp(HumansgramApp());
}

class HumansgramApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      title: 'Humansgram',
      theme: ThemeData.dark().copyWith(
        scaffoldBackgroundColor: Color(0xFF0A0A0A),
        textTheme: ThemeData.dark().textTheme.apply(fontFamily: 'Inter'),
      ),
      home: ChatListScreen(),
    );
  }
}

class ChatListScreen extends StatefulWidget {
  @override
  _ChatListScreenState createState() => _ChatListScreenState();
}

class _ChatListScreenState extends State<ChatListScreen> {
  final supabase = Supabase.instance.client;
  List chats = [];

  @override
  void initState() {
    super.initState();
    fetchChats();
    // Real-time - Telegram'dek jonli
    supabase.channel('chats').onPostgresChanges(
      event: PostgresChangeEvent.all,
      schema: 'public',
      table: 'chats',
      callback: (payload) => fetchChats(),
    ).subscribe();
  }

  Future<void> fetchChats() async {
    final data = await supabase.from('chats').select().order('created_at');
    setState(() => chats = data);
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: Color(0xFF0A0A0A),
      appBar: AppBar(
        backgroundColor: Colors.black,
        title: Row(
          children: [
            Container(
              width: 36, height: 36,
              decoration: BoxDecoration(
                color: Colors.white,
                borderRadius: BorderRadius.circular(10),
              ),
              child: Center(child: Text('H', style: TextStyle(color: Colors.black, fontWeight: FontWeight.bold, fontSize: 20))),
            ),
            SizedBox(width: 12),
            Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                Text('HUMANSGRAM', style: TextStyle(fontWeight: FontWeight.bold, fontSize: 16, letterSpacing: 1.2)),
                Text('Connect. Create. Communicate.', style: TextStyle(fontSize: 10, color: Colors.grey)),
              ],
            ),
          ],
        ),
      ),
      body: ListView.builder(
        itemCount: chats.length,
        itemBuilder: (context, i) {
          final chat = chats[i];
          return Container(
            margin: EdgeInsets.symmetric(horizontal: 12, vertical: 6),
            decoration: BoxDecoration(
              color: Colors.white.withOpacity(0.08),
              borderRadius: BorderRadius.circular(20),
              border: Border.all(color: Colors.white.withOpacity(0.12)),
            ),
            child: ListTile(
              leading: CircleAvatar(
                backgroundColor: Colors.blueAccent,
                child: Text(chat['name'][0], style: TextStyle(color: Colors.white)),
              ),
              title: Text(chat['name'], style: TextStyle(color: Colors.white, fontSize: 17, fontWeight: FontWeight.w500)),
              subtitle: Text(chat['type'], style: TextStyle(color: Colors.grey)),
              trailing: Text('now', style: TextStyle(color: Colors.grey, fontSize: 12)),
              onTap: () => Navigator.push(context, MaterialPageRoute(builder: (_) => ChatScreen(chat: chat))),
            ),
          );
        },
      ),
      floatingActionButton: FloatingActionButton(
        backgroundColor: Color(0xFF5B8CFF),
        child: Icon(Icons.add),
        onPressed: () => _showNewChatDialog(),
      ),
    );
  }

  void _showNewChatDialog() {
    showDialog(context: context, builder: (c) => AlertDialog(
      backgroundColor: Color(0xFF1A1A1A),
      title: Text('Yangi chat', style: TextStyle(color: Colors.white)),
      content: TextField(
        style: TextStyle(color: Colors.white),
        decoration: InputDecoration(hintText: 'Ism yozing...', hintStyle: TextStyle(color: Colors.grey)),
        onSubmitted: (name) async {
          await supabase.from('chats').insert({'type': 'private', 'name': name});
          Navigator.pop(c);
        },
      ),
    ));
  }
}

class ChatScreen extends StatefulWidget {
  final Map chat;
  ChatScreen({required this.chat});
  @override
  _ChatScreenState createState() => _ChatScreenState();
}

class _ChatScreenState extends State<ChatScreen> {
  final supabase = Supabase.instance.client;
  final controller = TextEditingController();
  List messages = [];

  @override
  void initState() {
    super.initState();
    fetchMessages();
    supabase.channel('messages-${widget.chat['id']}').onPostgresChanges(
      event: PostgresChangeEvent.insert,
      schema: 'public',
      table: 'messages',
      filter: PostgresChangeFilter(type: PostgresChangeFilterType.eq, column: 'chat_id', value: widget.chat['id']),
      callback: (p) => fetchMessages(),
    ).subscribe();
  }

  Future<void> fetchMessages() async {
    final data = await supabase.from('messages').select().eq('chat_id', widget.chat['id']).order('created_at');
    setState(() => messages = data);
  }

  Future<void> sendMessage() async {
    if (controller.text.isEmpty) return;
    final text = controller.text;
    
    if (text.startsWith('/ai')) {
      final q = text.replaceFirst('/ai', '').trim();
    }
    
    await supabase.from('messages').insert({
      'chat_id': widget.chat['id'],
      'text': text,
      'sender': 'me',
    });
    controller.clear();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: Color(0xFF0A0A0A),
      appBar: AppBar(backgroundColor: Colors.black, title: Text(widget.chat['name'])),
      body: Column(
        children: [
          Expanded(child: ListView.builder(
            itemCount: messages.length,
            itemBuilder: (c,i) {
              final m = messages[i];
              return Align(
                alignment: m['sender']=='me'? Alignment.centerRight : Alignment.centerLeft,
                child: Container(
                  margin: EdgeInsets.all(6),
                  padding: EdgeInsets.symmetric(horizontal: 14, vertical: 10),
                  decoration: BoxDecoration(
                    color: m['sender']=='me'? Color(0xFF5B8CFF) : Colors.white.withOpacity(0.08),
                    borderRadius: BorderRadius.circular(16),
                  ),
                  child: Text(m['text'], style: TextStyle(color: Colors.white)),
                ),
              );
            },
          )),
          Padding(
            padding: EdgeInsets.all(8),
            child: Row(children: [
              Expanded(child: TextField(controller: controller, style: TextStyle(color: Colors.white), decoration: InputDecoration(hintText: 'Xabar...', hintStyle: TextStyle(color: Colors.grey), filled: true, fillColor: Colors.white.withOpacity(0.08), border: OutlineInputBorder(borderRadius: BorderRadius.circular(24))))),
              SizedBox(width: 8),
              IconButton(icon: Icon(Icons.send, color: Colors.white), onPressed: sendMessage),
            ]),
          ),
        ],
      ),
    );
  }
}
