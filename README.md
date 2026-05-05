#include <iostream>
#include <vector>
#include <fstream>
#include <iomanip>
#include <ctime>
#include <cstdlib>
using namespace std;

class Stock
{
public:
    string symbol;
    string companyName;
    double price;

    Stock(string s, string c, double p)
    {
        symbol = s;
        companyName = c;
        price = p;
    }
};

class Holding
{
public:
    string symbol;
    int quantity;
    double buyPrice;

    Holding(string s, int q, double bp)
    {
        symbol = s;
        quantity = q;
        buyPrice = bp;
    }
};

class User
{
public:
    string username;
    string password;
    double balance;
    vector<Holding> portfolio;

    User(string u, string p, double b)
    {
        username = u;
        password = p;
        balance = b;
    }

    
    void buyStock(vector<Stock> &market)
    {
        string symbol;
        int quantity;

        cout << "\nEnter Stock Symbol to Buy: ";
        cin >> symbol;

        for (auto &stock : market)
        {
            if (stock.symbol == symbol)
            {
                cout << "Enter Quantity: ";
                cin >> quantity;

                double totalCost = stock.price * quantity;

                if (totalCost > balance)
                {
                    cout << "\n❌ Insufficient Balance!\n";
                    return;
                }

                balance -= totalCost;
                portfolio.push_back(Holding(symbol, quantity, stock.price));

                cout << "\n✅ Stock Purchased Successfully!\n";
                cout << "Remaining Balance: ₹" << balance << endl;

                saveTransaction("BUY", symbol, quantity, stock.price);
                return;
            }
        }

        cout << "\n❌ Stock Not Found!\n";
    }

    void sellStock(vector<Stock> &market)
    {
        string symbol;
        int quantity;

        cout << "\nEnter Stock Symbol to Sell: ";
        cin >> symbol;

        for (int i = 0; i < portfolio.size(); i++)
        {
            if (portfolio[i].symbol == symbol)
            {
                cout << "Enter Quantity to Sell: ";
                cin >> quantity;

                if (quantity > portfolio[i].quantity)
                {
                    cout << "\n❌ Not Enough Shares!\n";
                    return;
                }

                double currentPrice = 0;

                for (auto &stock : market)
                {
                    if (stock.symbol == symbol)
                    {
                        currentPrice = stock.price;
                        break;
                    }
                }

                double amount = currentPrice * quantity;
                balance += amount;
                portfolio[i].quantity -= quantity;

                if (portfolio[i].quantity == 0)
                {
                    portfolio.erase(portfolio.begin() + i);
                }

                cout << "\n✅ Stock Sold Successfully!\n";
                cout << "Updated Balance: ₹" << balance << endl;

                saveTransaction("SELL", symbol, quantity, currentPrice);
                return;
            }
        }

        cout << "\n❌ You Do Not Own This Stock!\n";
    }

    void viewPortfolio(vector<Stock> &market)
    {
        cout << "\n YOUR PORTFOLIO\n";

        if (portfolio.empty())
        {
            cout << "No Stocks Purchased Yet!\n";
            return;
        }

        double totalInvestment = 0;
        double currentValue = 0;

        cout << left << setw(10) << "Symbol"
             << setw(10) << "Qty"
             << setw(15) << "Buy Price"
             << setw(15) << "Current Price"
             << setw(15) << "Profit/Loss" << endl;

        for (auto &holding : portfolio)
        {
            double currentPrice = 0;

            for (auto &stock : market)
            {
                if (stock.symbol == holding.symbol)
                {
                    currentPrice = stock.price;
                    break;
                }
            }

            double invested = holding.buyPrice * holding.quantity;
            double current = currentPrice * holding.quantity;
            double profitLoss = current - invested;

            totalInvestment += invested;
            currentValue += current;

            cout << left << setw(10) << holding.symbol
                 << setw(10) << holding.quantity
                 << setw(15) << holding.buyPrice
                 << setw(15) << currentPrice
                 << setw(15) << profitLoss << endl;
        }

        cout << "\nTotal Investment: ₹" << totalInvestment << endl;
        cout << "Current Portfolio Value: ₹" << currentValue << endl;
        cout << "Net Profit/Loss: ₹" << currentValue - totalInvestment << endl;
    }

    void saveTransaction(string type, string symbol, int quantity, double price)
    {
        ofstream file("transactions.txt", ios::app);

        file << username << " | "
             << type << " | "
             << symbol << " | "
             << quantity << " | "
             << price << endl;

        file.close();
    }

    void viewTransactions()
    {
        ifstream file("transactions.txt");
        string line;

        cout << "\n============= TRANSACTION HISTORY =============\n";

        while (getline(file, line))
        {
            if (line.find(username) != string::npos)
            {
                cout << line << endl;
            }
        }

        file.close();
    }
};

void displayMarket(vector<Stock> &market)
{
    cout << "\n================ STOCK MARKET ================\n";

    cout << left << setw(10) << "Symbol"
         << setw(25) << "Company"
         << setw(10) << "Price" << endl;

    for (auto &stock : market)
    {
        cout << left << setw(10) << stock.symbol
             << setw(25) << stock.companyName
             << "₹" << stock.price << endl;
    }
}

void updateMarket(vector<Stock> &market)
{
    srand(time(0));

    for (auto &stock : market)
    {
        int fluctuation = rand() % 21 - 10;
        stock.price += fluctuation;

        if (stock.price < 50)
        {
            stock.price = 50;
        }
    }
}

int main()
{
    vector<Stock> market;

    market.push_back(Stock("AAPL", "Apple Inc.", 180));
    market.push_back(Stock("TSLA", "Tesla", 250));
    market.push_back(Stock("GOOGL", "Google", 140));
    market.push_back(Stock("NVDA", "NVIDIA", 450));
    market.push_back(Stock("AMZN", "Amazon", 160));

    string username, password;

    cout << "===============================================\n";
    cout << "   STOCK MARKET PORTFOLIO MANAGEMENT SYSTEM\n";
    cout << "===============================================\n";

    cout << "\nCreate Username: ";
    cin >> username;

    cout << "Create Password: ";
    cin >> password;

    User currentUser(username, password, 100000);

    int choice;

    do
    {
        updateMarket(market);

        cout << "\n===============================================\n";
        cout << "1. View Market\n";
        cout << "2. Buy Stock\n";
        cout << "3. Sell Stock\n";
        cout << "4. View Portfolio\n";
        cout << "5. View Transaction History\n";
        cout << "6. View Balance\n";
        cout << "7. Exit\n";
        cout << "===============================================\n";

        cout << "Enter Choice: ";
        cin >> choice;

        switch (choice)
        {
        case 1:
            displayMarket(market);
            break;

        case 2:
            displayMarket(market);
            currentUser.buyStock(market);
            break;

        case 3:
            currentUser.sellStock(market);
            break;

        case 4:
            currentUser.viewPortfolio(market);
            break;

        case 5:
            currentUser.viewTransactions();
            break;

        case 6:
            cout << "\nCurrent Balance: ₹" << currentUser.balance << endl;
            break;

        case 7:
            cout << "\n✅ Thank You For Using The System!\n";
            break;

        default:
            cout << "\n❌ Invalid Choice!\n";
        }

    } while (choice != 7);

    return 0;
}
