안드로이드 앱 내에서 블루투스 페어링 및 연결을 위해 AlertDialog, Recyclerview를 사용하는 예시 문서이다.

#### Permission
우선, 블루투스 관련 기능을 사용하기 위해 권한 설정부터 진행한다.

```AndroidManifest.xml
 <!-- Bluetooth -->
    <uses-permission android:name="android.permission.BLUETOOTH" />
    <uses-permission android:name="android.permission.BLUETOOTH_ADMIN" />
    <uses-permission android:name="android.permission.BLUETOOTH_CONNECT" />
    <uses-permission android:name="android.permission.BLUETOOTH_SCAN"
        android:usesPermissionFlags="neverForLocation"
        tools:targetApi="s" />
```

#### RecyclerView 기본 인터페이스 구현

RecyclerView는 데이터를 리스트 형태로 보여 주고, 리소스를 회수하여 효율적 뷰 구성을 도와준다. 리사이클러뷰를 사용하기 위해, 기본 인터페이스로 아래 항목을 구현해야 한다.

1. **RecyclerView Adapter**: 데이터를 RecyclerView에 바인딩
2. **ViewHolder**: RecyclerView의 각 항목을 보유하고 재활용
3. **Item Layout**: 각 항목의 레이아웃(디자인) 정의
#### Adapter 클래스 정의
Adapter 클래스는 RecyclerView에 데이터를 공급하고, 각 항목을 ViewHolder에 바인딩하는 역할을 한다. BluetoothDeviceAdapter는 블루투스 기기 목록을 RecyclerView에 표시하기 위해 사용된다.
```BluetoothDeviceAdapter.kt
package com.example.bt_test

import android.bluetooth.BluetoothDevice
import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import android.widget.TextView
import androidx.recyclerview.widget.RecyclerView

class BluetoothDeviceAdapter(
    private val devices: List<BluetoothDevice>,
    private val onItemClickListener: (BluetoothDevice) -> Unit
) : RecyclerView.Adapter<BluetoothDeviceAdapter.DeviceViewHolder>() {

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): DeviceViewHolder {
        val view = LayoutInflater.from(parent.context).inflate(android.R.layout.simple_list_item_2, parent, false)
        return DeviceViewHolder(view)
    }

    override fun onBindViewHolder(holder: DeviceViewHolder, position: Int) {
        val device = devices[position]
        holder.nameTextView.text = device.name
        holder.addressTextView.text = device.address
        holder.itemView.setOnClickListener { onItemClickListener(device) }
    }

    override fun getItemCount(): Int {
        return devices.size
    }

    class DeviceViewHolder(itemView: View) : RecyclerView.ViewHolder(itemView) {
        val nameTextView: TextView = itemView.findViewById(android.R.id.text1)
        val addressTextView: TextView = itemView.findViewById(android.R.id.text2)
    }
}
```

#### MainActivity
위에서 구성한 어댑터 클래스를 이용해, AlertDialog 내부에 RecyclerView가 위치하도록 한다. MainActivity는 다음과 같이 구성하였다.

- 버튼을 누르면 AlertDialog가 나오며, 이미 연결된 블루투스 기기는 처음부터 RecyclerView에 목록으로 존재
- AlertDialog의 버튼 'Search'를 누르면, 연결 가능한 새로운 블루투스 기기를 검색
- AlertDialog의 버튼 'Cancel'을 누르면, 다이얼로그를 닫음
- RecyclerView의 목록을 터치하면, 해당 블루투스 기기에 페어링 + 연결을 시도함
- 연결에 성공할 경우, MainActivity의 TextView에 해당 기기의 정보를 출력함

![[Pasted image 20240611171419.png]]
> 초기 Activity 화면

![[Pasted image 20240611171447.png]]
> AlertDialog, 기존 기기에 연결 성공

![[Pasted image 20240611171558.png]]
> 'Scan' 버튼 입력 시

```MainActivitiy.kt

class MainActivity : AppCompatActivity() {

    private lateinit var textView: TextView
    private lateinit var button: Button
    private lateinit var bluetoothAdapter: BluetoothAdapter
    private lateinit var deviceAdapter: BluetoothDeviceAdapter
    private val deviceList = ArrayList<BluetoothDevice>()

    private val multiplePermissionsLauncher =
        registerForActivityResult(ActivityResultContracts.RequestMultiplePermissions()) { permissions ->
            val deniedPermissions = permissions.filter { !it.value }
            if (deniedPermissions.isEmpty()) {
                // 모든 권한이 허용된 경우
                Toast.makeText(this, "모든 권한 허용됨", Toast.LENGTH_SHORT).show()
                enableBluetooth()
            } else {
                // 권한이 거부된 경우 안내 메시지 표시
                showPermissionDeniedDialog(deniedPermissions.keys.joinToString(", "))
            }
        }

    private val enableBtLauncher =
        registerForActivityResult(ActivityResultContracts.StartActivityForResult()) { result ->
            if (result.resultCode == RESULT_OK) {
                showDeviceDialog()
            } else {
                Toast.makeText(this, "블루투스 권한 혹은 연결이 필요합니다.", Toast.LENGTH_SHORT).show()
            }
        }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        textView = findViewById(R.id.textView)
        button = findViewById(R.id.button)

        bluetoothAdapter = BluetoothAdapter.getDefaultAdapter()
        deviceAdapter = BluetoothDeviceAdapter(deviceList) { device -> pairDevice(device) }

        button.setOnClickListener { checkPermissionsAndEnableBluetooth() }

        // Register for broadcasts when a device is discovered and bond state changes
        val filter = IntentFilter(BluetoothDevice.ACTION_FOUND)
        filter.addAction(BluetoothDevice.ACTION_BOND_STATE_CHANGED)
        registerReceiver(receiver, filter)
    }

    @RequiresApi(Build.VERSION_CODES.S)
    private fun checkPermissionsAndEnableBluetooth() {
        val permissionsToRequest = mutableListOf<String>()

        if (ContextCompat.checkSelfPermission(this, Manifest.permission.BLUETOOTH_SCAN) != PackageManager.PERMISSION_GRANTED) {
            permissionsToRequest.add(Manifest.permission.BLUETOOTH_SCAN)
        }

        if (ContextCompat.checkSelfPermission(this, Manifest.permission.BLUETOOTH_CONNECT) != PackageManager.PERMISSION_GRANTED) {
            permissionsToRequest.add(Manifest.permission.BLUETOOTH_CONNECT)
        }

        if (permissionsToRequest.isNotEmpty()) {
            multiplePermissionsLauncher.launch(permissionsToRequest.toTypedArray())
        } else {
            enableBluetooth()
        }
    }

    private fun enableBluetooth() {
        if (!bluetoothAdapter.isEnabled) {
            val enableBtIntent = Intent(BluetoothAdapter.ACTION_REQUEST_ENABLE)
            enableBtLauncher.launch(enableBtIntent)
        } else {
            showDeviceDialog()
        }
    }

    private fun startBluetoothConnection() {
        val pairedDevices: Set<BluetoothDevice>? = bluetoothAdapter.bondedDevices
        if (!pairedDevices.isNullOrEmpty()) {
            val device = pairedDevices.first() // 첫 번째 페어링된 기기로 연결 시도
            textView.text = "Connected to: ${device.name} (${device.address})"
        } else {
            textView.text = "블루투스 기기를 찾지 못했습니다."
        }
    }

    private fun showPermissionDeniedDialog(deniedPermissions: String) {
        val intent = Intent(Settings.ACTION_APPLICATION_DETAILS_SETTINGS).apply {
            data = Uri.fromParts("package", packageName, null)
        }

        // Intent 값 로그로 출력
        Log.d("MainActivity", "Intent to settings: $intent")

        AlertDialog.Builder(this)
            .setTitle("접근 권한 필요")
            .setMessage("다음 권한이 접근 거부되었습니다: $deniedPermissions. 접근 권한을 허용해 주세요.")
            .setPositiveButton("권한 설정") { _, _ ->
                // 사용자를 설정 화면으로 이동
                startActivity(intent)
            }
            .setNegativeButton("종료하기") { _, _ ->
                // 다이얼로그 닫기
                finish()
            }
            .show()
    }

    private fun showDeviceDialog() {
        val builder = AlertDialog.Builder(this)
        builder.setTitle("Select a device")

        val recyclerView = RecyclerView(this).apply {
            layoutManager = LinearLayoutManager(this@MainActivity)
            adapter = deviceAdapter
            layoutParams = LayoutParams(LayoutParams.MATCH_PARENT, 600) // RecyclerView 높이 제한
        }

        builder.setView(recyclerView)
        builder.setPositiveButton("Scan", null) // 버튼 클릭 시 다이얼로그가 닫히지 않도록 null 처리
        builder.setNegativeButton("Cancel") { dialog, _ -> dialog.dismiss() }

        val dialog = builder.create()
        dialog.show()

        // Scan 버튼을 별도로 설정
        dialog.getButton(AlertDialog.BUTTON_POSITIVE).setOnClickListener {
            discoverDevices()
        }

        // Show paired devices
        val pairedDevices: Set<BluetoothDevice>? = bluetoothAdapter.bondedDevices
        deviceList.clear()
        pairedDevices?.let {
            deviceList.addAll(it)
            deviceAdapter.notifyDataSetChanged()
        }
    }

    private fun discoverDevices() {
        if (bluetoothAdapter.isDiscovering) {
            bluetoothAdapter.cancelDiscovery()
        }
        bluetoothAdapter.startDiscovery()
    }

    private fun pairDevice(device: BluetoothDevice) {
        if (device.bondState == BluetoothDevice.BOND_NONE) {
            device.createBond()
        } else {
            connectDevice(device)
        }
    }

    private fun connectDevice(device: BluetoothDevice) {
        // UUID 확인
        val uuid = device.uuids?.get(0)?.uuid ?: return
        var bluetoothSocket: BluetoothSocket? = null

        try {
            bluetoothSocket = device.createRfcommSocketToServiceRecord(uuid)
            bluetoothAdapter.cancelDiscovery()
            bluetoothSocket.connect()
            runOnUiThread {
                textView.text = "Connected to: ${device.name}\n${device.address}"
                Toast.makeText(this, "Connected to ${device.name}", Toast.LENGTH_SHORT).show()
            }
        } catch (e: IOException) {
            e.printStackTrace()
            runOnUiThread {
                Toast.makeText(this, "Failed to connect to ${device.name}", Toast.LENGTH_SHORT).show()
            }
            try {
                bluetoothSocket?.close()
            } catch (closeException: IOException) {
                closeException.printStackTrace()
            }
        }
    }


    private val receiver = object : BroadcastReceiver() {
        override fun onReceive(context: Context, intent: Intent) {
            val action: String = intent.action ?: return
            if (BluetoothDevice.ACTION_FOUND == action) {
                val device: BluetoothDevice =
                    intent.getParcelableExtra(BluetoothDevice.EXTRA_DEVICE)!!
                if (deviceList.none { it.address == device.address }) {
                    deviceList.add(device)
                    deviceAdapter.notifyDataSetChanged()
                }
            } else if (BluetoothDevice.ACTION_BOND_STATE_CHANGED == action) {
                val device: BluetoothDevice =
                    intent.getParcelableExtra(BluetoothDevice.EXTRA_DEVICE)!!
                if (device.bondState == BluetoothDevice.BOND_BONDED) {
                    connectDevice(device)
                }
            }
        }
    }

    override fun onDestroy() {
        super.onDestroy()
        unregisterReceiver(receiver)
    }
}
```