- 👋 Hi, I’m @ANDERSONPTO
- 👀 I’m interested in ...
- 🌱 I’m currently learning ...
- 💞️ I’m looking to collaborate on ...
- 📫 How to reach me ...

<!---
ANDERSONPTO/ANDERSONPTO is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
input int ema_curta_periodo = 14; // Período da Média Móvel Curta
input int ema_longa_periodo = 50; // Período da Média Móvel Longa
input int macd_periodo_curto = 12; // Período Curto do MACD
input int macd_periodo_longo = 26; // Período Longo do MACD
input int macd_periodo_sinal = 9; // Período do Sinal do MACD
input double tamanho_lote = 0.1; // Tamanho do Lote
input int slippage = 3; // Slippage
input int lucro_alvo_pontos = 100; // Alvo de lucro em pontos

int handle_ema_curta, handle_ema_longa, handle_macd;
double ema_curta_buffer[], ema_longa_buffer[], macd_main_buffer[], macd_signal_buffer[];
int arrow_buy, arrow_sell;
double preco_compra, preco_venda;

//+------------------------------------------------------------------+
//| Expert initialization function                                   |
//+------------------------------------------------------------------+
int OnInit()
{
   handle_ema_curta = iMA(Symbol(), 0, ema_curta_periodo, 0, MODE_EMA, PRICE_CLOSE);
   handle_ema_longa = iMA(Symbol(), 0, ema_longa_periodo, 0, MODE_EMA, PRICE_CLOSE);
   handle_macd = iMACD(Symbol(), 0, macd_periodo_curto, macd_periodo_longo, macd_periodo_sinal, PRICE_CLOSE);

   ArraySetAsSeries(ema_curta_buffer, true);
   ArraySetAsSeries(ema_longa_buffer, true);
   ArraySetAsSeries(macd_main_buffer, true);
   ArraySetAsSeries(macd_signal_buffer, true);

   arrow_buy = iCustom(Symbol(), 0, "Gadu", "ArrowBuy", 0, 1);
   arrow_sell = iCustom(Symbol(), 0, "Gadu", "ArrowSell", 0, 1);

   return(INIT_SUCCEEDED);
}

//+------------------------------------------------------------------+
//| Expert deinitialization function                                 |
//+------------------------------------------------------------------+
void OnDeinit(const int reason)
{
   ObjectsDeleteAll(0, OBJ_ARROW);
}

//+------------------------------------------------------------------+
//| Expert tick function                                             |
//+------------------------------------------------------------------+
void OnTick()
{
   if (iBars(Symbol(), 0) <= ema_longa_periodo)
      return;

   double ema_curta_valor = iMA(Symbol(), 0, ema_curta_periodo, 0, MODE_EMA, PRICE_CLOSE, 0);
   double ema_longa_valor = iMA(Symbol(), 0, ema_longa_periodo, 0, MODE_EMA, PRICE_CLOSE, 0);
   double macd_valor = iMACD(Symbol(), 0, macd_periodo_curto, macd_periodo_longo, macd_periodo_sinal, PRICE_CLOSE, 0);

   CopyBuffer(handle_ema_curta, 0, 0, ema_curta_periodo, ema_curta_buffer);
   CopyBuffer(handle_ema_longa, 0, 0, ema_longa_periodo, ema_longa_buffer);
   CopyBuffer(handle_macd, 0, 0, macd_periodo_sinal, macd_signal_buffer);
   CopyBuffer(handle_macd, 1, 0, macd_periodo_sinal, macd_main_buffer);

   // Lógica de compra
   if (ema_curta_valor > ema_longa_valor && macd_valor > 0 && ema_curta_buffer[0] > ema_longa_buffer[0] && macd_main_buffer[0] > macd_signal_buffer[0])
   {
      if (arrow_buy == 1)
      {
         ObjectCreate(0, "BuyArrow", OBJ_ARROW, 0, Time[0], Low[0] - Point);
         ObjectSetInteger(0, "BuyArrow", OBJPROP_ARROWCODE, 233);
         ObjectSetInteger(0, "BuyArrow", OBJPROP_COLOR, clrRed); // Corrigido: vermelho para compra
         ObjectSetInteger(0, "BuyArrow", OBJPROP_WIDTH, 2);
         preco_compra = Ask;
      }
      if (preco_compra + lucro_alvo_pontos * Point <= Bid)
      {
         OrderClose(OrderSend(Symbol(), OP_BUY, tamanho_lote, preco_compra, slippage, 0, 0, "", 0, clrNONE), tamanho_lote, Bid, slippage, clrNONE);
      }
   }

   // Lógica de venda
   else if (ema_curta_valor < ema_longa_valor && macd_valor < 0 && ema_curta_buffer[0] < ema_longa_buffer[0] && macd_main_buffer[0] < macd_signal_buffer[0])
   {
      if (arrow_sell == 1)
      {
         ObjectCreate(0, "SellArrow", OBJ_ARROW, 0, Time[0], High[0] + Point);
         ObjectSetInteger(0, "SellArrow", OBJPROP_ARROWCODE, 234);
         ObjectSetInteger(0, "SellArrow", OBJPROP_COLOR, clrLime); // Corrigido: verde para venda
         ObjectSetInteger(0, "SellArrow", OBJPROP_WIDTH, 2);
         preco_venda = Bid;
      }
      if (preco_venda - lucro_alvo_pontos * Point >= Ask)
      {
         OrderClose(OrderSend(Symbol(), OP_SELL, tamanho_lote, preco_venda, slippage, 0, 0, "", 0, clrNONE), tamanho_lote, Ask, slippage, clrNONE);
      }
   }
}
