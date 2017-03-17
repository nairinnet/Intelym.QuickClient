# Intelym.QuickClient
.Net Market Data Client API to Quick Application

# namespaces to be imported in the caller application
```c#
Intelym.QuickClient.Services
Intelym.QuickClient.Data
```

# interface to be implemented by caller application
```c#
Intelym.QuickClient.Services.QuickEvent

// raised in the event of successful connection with Quick Server
void OnConnect();
// raised in the event on forceful disconnection
void OnDisconnect(EventDetails details);
// raised in the event of any error, implied disconnection also
void OnError(EventDetails details);
// raised when single packet arrives
void OnPacketArrived(Packet packet);
// raised when multiple packet arrives
void OnPacketArrived(Packet[] packet);
```
# EventDetails
Event details is a holder for event description and codes occur with in the quickclient
Contains two properties Code and Description.

Code will carry one of the below EventIds
```c#
int FORCEFUL_DISCONNECTION = 101; /// when internal disconnection occurs
int EXTERNAL_DISCONNECTION = 201; /// when external disconnection occurs
int INTERPRET_ERROR = 202; /// if any failure in packet interpretation
int PACKETPROCESSING_FAILED = 203; /// if any failure in packet interpretation
int DECODING_FAILED = 204; /// when the base64 decoding fails
int UNABLE_TO_CONNECT = 301/ /// when unable to connect to Quick server
```
# Packet
Packet class is an top most parent class for all the type of packets, 
there are four known subclasses, see at the end of this document for properties of the subclasses
* IndexPacket
* QuotePacket
* MarketDepthPacket
* DerivativePacket
* OpenInterestPacket
* NewsPacket

# Client instance and properties
The caller application is expected to get the instance of Handler and then access its properties and methods
```c#
private Handler _handler = MarketData.GetInstance();
```

Sets the event handler for callback events
```c#
SetEventHandler(QuickEvent event)
```
Sets the server ip address/domain of Quick server
```c#
SetAddress(string address);
```
Sets the port no of Quick server
```c#
SetPort(int port)
```
Sets the user credentials, password can be blank if server does not do any validation
```c#
SetUserCredentials(string username, string password)
```
Sets the multicast group and port details, use only if receiving data is on multicast, otherwise do not use this method
```c#
SetMulticastDetails(string mGroup, int mPort)
```
If logging is required, call the below function with logging path
```c#
SetEnableLogging(string logPath)
```
Following methods are used to register/unregister scrips for quote, marketdepth and derivative chain
```c#
bool AddScrip(int exchange, int scripCode)
bool AddScrip(int exchange, int[] scripCode)
bool AddMarketDepth(int exchange, int scripCode)
bool DeleteMarketDepth(int exchange, int scripCode)
bool DeleteScrip(int exchange, int scripCode)
bool AddDerivativeChain(int exchange, int scripCode) // underlying scripcode
bool DeleteDerivativeChain(int exchange, int scripCode) // underlying scripcode
```
Exchange codification
```c#
Exchanges.NSE = 0
Exchanges.BSE = 1
Exchanges.NSEFO = 2,
Exchanges.NSECUR = 3,
Exchanges.NCDEX = 4,
Exchanges.MCX = 5
```

Connecting and Disconnecting
```c#
bool Connect() // callback events of OnConnect() or OnError() will occur
bool Disconnect() // no callback, 100% disconnection
```
Enabling exchange news, set this method to true only if you want to receive exchange news, default is false
```c#
SetExchangeBroadcast(bool enable)
```

# Usage
```c#
using Intelym.QuickClient.Services;
using Intelym.QuickClient.Data;

class Caller : QuickEvent
{
    private Handler handler;
    public Caller()
    {
        try { 
            handler = MarketData.GetInstance();
            handler.setEventHandler(this);
            handler.SetAddress("serveraddress");
            handler.SetPort(serverport);
            handler.SetMulticastDetails("236.0.0.1", 6700);
            handler.SetUserCredentials("xxxx", "xxxxx");
            if (handler.Connect())
            {
                System.Console.WriteLine("Connect initiated");
            }
            else
            {
                System.Console.WriteLine("Connect failed");
            }
        }catch(Exception e)
        {
            System.Console.WriteLine(e.Message);
        }
    }
    static void Main(string[] args)
    {
        new Caller();
    }

    public void OnConnect()
    {
        System.Console.WriteLine("Connect succeeded");
        handler.AddScrip(Exchanges.BSE, 500325);
    }

    public void OnDisconnect(EventDetails details)
    {
        System.Console.WriteLine("Disconnect succeeded");
    }

    public void OnError(EventDetails details)
    {
        System.Console.WriteLine(details.Code);
    }

    public void OnPacketArrived(Packet[] packet)
    {
       System.Console.WriteLine("Packet Arrived");
    }

    public void OnPacketArrived(Packet packet)
    {
        if (packet is IndexPacket) 
        {
            System.Console.WriteLine("Index Packet Arrived");
        }
        else if (packet is QuotePacket)
        {
            System.Console.WriteLine("Quote Packet Arrived");
        }
        else if (packet is MarketDepthPacket)
        {
            System.Console.WriteLine("Market Depth Packet Arrived");
        }
        else if (packet is DerivativePacket)
        {
            System.Console.WriteLine("Derivative Packet Arrived");
        }
        else if (packet is OpenInterestPacket)
        {
            System.Console.WriteLine("Open Interest Packet Arrived");
        }
    }
}
```

# Packet Properties
Packet
```c#
 int ScripCode 
 int Exchange
 int Timestamp
 int Session
```
IndexPacket
```c#
 int LastTradedPrice
 int ClosePrice
```
QuotePacket
```c#
 int LastTradedPrice 
 int ClosePrice 
 int BestBuyPrice 
 int BestBuyQty 
 int BestSellPrice 
 int BestSellQty 
 int HighPrice 
 int LowPrice 
 int OpenPrice 
 int LastTradedQty 
 int TotalTradedQty 
 int WeightedAveragePrice 
 int TotalBuy 
 int TotalSell 
 int TotalTradedValue 
```
MarketDepthPacket
```c#
 int NoOfRecords 
 int[] BuyPrice 
 int[] SellPrice 
 int[] BuyQty 
 int[] SellQty 
 int[] BuyOrders 
 int[] SellOrders 
```
DerivativePacket 
```c#
 int UnderlyingScripCode
All of the fields of Quote Packet
```
OpenInterestPacket
```c#
 int OpenInterest
 int LastTradedPrice
 int TotalTradedQty
```
