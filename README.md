# Controle-Financeiro-1.1
import 'package:flutter/material.dart';

void main() {
  runApp(ControleGastosApp());
}

class ControleGastosApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Controle de Gastos',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: HomeScreen(),
    );
  }
}

class Gasto {
  final String descricao;
  final double valor;
  final DateTime data;

  Gasto({required this.descricao, required this.valor, required this.data});
}

class HomeScreen extends StatefulWidget {
  @override
  _HomeScreenState createState() => _HomeScreenState();
}

class _HomeScreenState extends State<HomeScreen> {
  List<Gasto> gastos = [];

  void _adicionarGasto(Gasto gasto) {
    setState(() {
      gastos.add(gasto);
    });
  }

  double _calcularTotal() {
    return gastos.fold(0.0, (soma, gasto) => soma + gasto.valor);
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Controle de Gastos'),
      ),
      body: Column(
        children: [
          Padding(
            padding: EdgeInsets.all(16.0),
            child: Text(
              'Total: R\$ ${_calcularTotal().toStringAsFixed(2)}',
              style: TextStyle(fontSize: 20, fontWeight: FontWeight.bold),
            ),
          ),
          Expanded(
            child: gastos.isEmpty
                ? Center(child: Text('Nenhum gasto adicionado.'))
                : ListView.builder(
                    itemCount: gastos.length,
                    itemBuilder: (context, index) {
                      final gasto = gastos[index];
                      return ListTile(
                        title: Text(gasto.descricao),
                        subtitle: Text(
                            'R\$ ${gasto.valor.toStringAsFixed(2)} - ${gasto.data.day}/${gasto.data.month}/${gasto.data.year}'),
                      );
                    },
                  ),
          ),
        ],
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () {
          Navigator.push(
            context,
            MaterialPageRoute(
              builder: (context) => AdicionarGastoScreen(
                onAdicionar: _adicionarGasto,
              ),
            ),
          );
        },
        child: Icon(Icons.add),
      ),
    );
  }
}

class AdicionarGastoScreen extends StatefulWidget {
  final Function(Gasto) onAdicionar;

  AdicionarGastoScreen({required this.onAdicionar});

  @override
  _AdicionarGastoScreenState createState() => _AdicionarGastoScreenState();
}

class _AdicionarGastoScreenState extends State<AdicionarGastoScreen> {
  final _formKey = GlobalKey<FormState>();
  String _descricao = '';
  double _valor = 0.0;
  DateTime _data = DateTime.now();

  void _selecionarData() async {
    final DateTime? picked = await showDatePicker(
      context: context,
      initialDate: _data,
      firstDate: DateTime(2000),
      lastDate: DateTime(2101),
    );
    if (picked != null && picked != _data) {
      setState(() {
        _data = picked;
      });
    }
  }

  void _salvarGasto() {
    if (_formKey.currentState!.validate()) {
      _formKey.currentState!.save();
      widget.onAdicionar(Gasto(descricao: _descricao, valor: _valor, data: _data));
      Navigator.pop(context);
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Adicionar Gasto'),
      ),
      body: Padding(
        padding: EdgeInsets.all(16.0),
        child: Form(
          key: _formKey,
          child: Column(
            children: [
              TextFormField(
                decoration: InputDecoration(labelText: 'Descrição'),
                validator: (value) {
                  if (value == null || value.isEmpty) {
                    return 'Por favor, insira uma descrição';
                  }
                  return null;
                },
                onSaved: (value) {
                  _descricao = value!;
                },
              ),
              TextFormField(
                decoration: InputDecoration(labelText: 'Valor (R\$)'),
                keyboardType: TextInputType.number,
                validator: (value) {
                  if (value == null || value.isEmpty) {
                    return 'Por favor, insira um valor';
                  }
                  if (double.tryParse(value) == null || double.parse(value) <= 0) {
                    return 'Insira um valor válido';
                  }
                  return null;
                },
                onSaved: (value) {
                  _valor = double.parse(value!);
                },
              ),
              SizedBox(height: 16),
              Row(
                children: [
                  Text(
                    'Data: ${_data.day}/${_data.month}/${_data.year}',
                    style: TextStyle(fontSize: 16),
                  ),
                  Spacer(),
                  ElevatedButton(
                    onPressed: _selecionarData,
                    child: Text('Selecionar Data'),
                  ),
                ],
              ),
              SizedBox(height: 20),
              ElevatedButton(
                onPressed: _salvarGasto,
                child: Text('Salvar'),
              ),
            ],
          ),
        ),
      ),
    );
  }
}
