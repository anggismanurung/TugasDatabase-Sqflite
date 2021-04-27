# TugasDatabase-Sqflite

sqflite: ^1.3.2+3

TransaksiPenjualan.dart

class TransaksiPenjualan {
  TransaksiPenjualan({
    this.namepelanggan,
    this.barang,
    this.jumlah,
    this.harga,
  });

  String namapelanggan;
  String barang;
  String jumlah;
  int harga;
}

MySqflite.dart

import 'package:explore_data/useCase/sqflite/TransaksiPenjualan.dart';
import 'package:path/path.dart';
import 'package:sqflite/sqflite.dart';

class MySqflite {
  static final _databaseName = "MyDatabase.db";

  static final _databaseV1 = 1;
  static final tableTransaksi = 'transaksi';

  static final columnNamaPelanggan = 'namapelanggan';
  static final columnBarang = 'barang';
  static final columnDJumlah = 'jumlah';
  static final columnHarga = 'harga';

  // make this a singleton class
  MySqflite._privateConstructor();

  static final MySqflite instance = MySqflite._privateConstructor();

  static Database _database;

  Future<Database> get database async {
    if (_database != null) return _database;
    _database = await _initDatabase();
    return _database;
  }

  _initDatabase() async {
    var databasesPath = await getDatabasesPath();
    String path = join(databasesPath, _databaseName);

    return await openDatabase(path, version: _databaseV1,
        onCreate: (db, version) async {
      var batch = db.batch();
      _onCreateTableTransaksi(batch);

      await batch.commit();
    });
  }

  void _onCreateTableTransaksi(Batch batch) async {
    batch.execute('''
          CREATE TABLE $tableTransaksi (
            $columnNamaPelanggan VARCHAR PRIMARY KEY,
            $columnBarang VARCHAR,
            $columnDJumlah VARCHAR,
            $columnHarga INTEGER
          )
          ''');
  }

  ///TABLE TRANSAKSI
  Future<int> insertTransaksi(getMTransaksiPenjualan transaksi) async {
    var row = {
      columnNamaPelanggan: transaksi.namapelanggan,
      columnBarang: transaksi.barang,
      columnJumlah: transaksi.jumlah,
      columnHarga: transaksi.harga    };

    Database db = await instance.database;
    return await db.insert(tableTransaksi, row);
  }

  Future<List<TransaksiPenjualan>> getMTransaksi() async {
    Database db = await instance.database;
    var allData = await db.rawQuery("SELECT * FROM $tableTransaksi");

    List<TransaksiPenjualan> result = [];
    for (var data in allData) {
      result.add(TransaksiPenjualan(
          namapelanggan: data[columnNamaPelanggan],
          barang: data[columnBarang],
          jumlah: data[columnJumlah],
          sks: int.parse(data[columnHarga].toString())));
    }

    return result;
  }

  Future<MahasiswaModel> getTransaksiByNAMAPELANGGAN(String barang) async {
    Database db = await instance.database;
    var allData = await db.rawQuery(
        "SELECT * FROM $tableTransaksi WHERE $columnNamaPelanggan = $namapelanggan LIMIT 1");

    if (allData.isNotEmpty) {
      return MahasiswaModel(
          nim: allData[0][columnNamaPelanggan],
          name: allData[0][columnBarang],
          department: allData[0][columnJumlah],
          sks: int.parse(allData[0][columnHarga]));
    } else {
      return null;
    }
  }

  Future<int> updateTransaksiJumlah(TransaksiPenjualan penjualan) async {
    Database db = await instance.database;
    return await db.rawUpdate(
        'UPDATE $tableTransaksi SET $columnJumlah = ${transaksi.jumlah} '
        'Where $columnnamepelanggan = ${model.namapelanggan}');
  }

  Future<int> deleteTransaksi(String namapelanggan) async {
    Database db = await instance.database;
    return await db
        .rawDelete('DELETE FROM $tableTransaksi Where $columnNamaPelanggan = $namapelanggan');
  }

  clearAllData() async {
    Database db = await instance.database;
    await db.rawQuery("DELETE FROM $tableTransaksi");
  }

}

SqfliteActivity.dart
import 'package:explore_data/useCase/sqflite/TransaksiPenjualan.dart';
import 'package:explore_data/useCase/sqflite/MySqlflite.dart';
import 'package:flutter/material.dart';

class SqfliteActivity extends StatefulWidget {
  @override
  State<StatefulWidget> createState() => SqfliteActivityState();
}

class SqfliteActivityState extends State<SqfliteActivity> {
  final keyFormTransaksi = GlobalKey<FormState>();

  TextEditingController controllerNamaPelanggan = TextEditingController();
  TextEditingController controllerBarang = TextEditingController();
  TextEditingController controllerJumlah = TextEditingController();
  TextEditingController controllerHarga= TextEditingController();

  String namapelanggan = "";
  String barang = "";
  String jumlah = "";
  int harga = 0;

  List<TransaksiPenjualan> transaksi = [];

  @override
  void initState() {
    super.initState();

    WidgetsBinding.instance.addPostFrameCallback((_) async {
      mahasiswa = await MySqflite.instance.getMTransaksi();
      setState(() {});
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
        body: Column(
      crossAxisAlignment: CrossAxisAlignment.stretch,
      children: [
        Container(
          margin: EdgeInsets.only(top: 36, left: 24, bottom: 4),
          child: Text("Input TransaksiPenjualan",
              style: TextStyle(fontWeight: FontWeight.bold)),
        ),
        Form(
          key: keyFormNamaPelanggan,
          child: Container(
            margin: EdgeInsets.only(left: 24, right: 24),
            child: Column(
              children: [
                TextFormField(
                  controller: controllerNamaPelanggan,
                  decoration: InputDecoration(hintText: "NamaPelanggan"),
                  validator: (value) => _onValidateText(value),
                  onSaved: (value) => namapelanggan = value,
                ),
                TextFormField(
                  controller: controllerBarang,
                  decoration: InputDecoration(hintText: "Barang"),
                  validator: (value) => _onValidateText(value),
                  onSaved: (value) => barang = value,
                ),
                TextFormField(
                  controller: controllerjumlah,
                  decoration: InputDecoration(hintText: "Jumlah"),
                  validator: (value) => _onValidateText(value),
                  onSaved: (value) => jumlah = value,
                ),
                TextFormField(
                  controller: controllerHarga,
                  decoration: InputDecoration(hintText: "Harga"),
                  validator: (value) => _onValidateText(value),
                  keyboardType: TextInputType.number,
                  onSaved: (value) => harga = int.parse(value),
                ),
              ],
            ),
          ),
        ),
        Container(
          margin: EdgeInsets.only(left: 24, right: 24),
          child: RaisedButton(
            onPressed: () {
              _onSaveTransaksi();
            },
            child: Text("Simpan"),
          ),
        ),
        Container(
          margin: EdgeInsets.only(top: 24, left: 24, bottom: 4),
          child: Text("Data Transaksi",
              style: TextStyle(fontWeight: FontWeight.bold)),
        ),
        Expanded(
            child: ListView.builder(
                itemCount: Transaksi.length,
                padding: EdgeInsets.fromLTRB(24, 0, 24, 8),
                itemBuilder: (BuildContext context, int index) {
                  var value = Transaksi[index];
                  return Container(
                    margin: EdgeInsets.only(bottom: 12),
                    child: Column(
                      crossAxisAlignment: CrossAxisAlignment.stretch,
                      mainAxisSize: MainAxisSize.min,
