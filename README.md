import React, { useState, useEffect } from "react";
import axios from "axios";
import { Card, CardContent } from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";

const WalletApp = () => {
  const [balance, setBalance] = useState(0);
  const [currency, setCurrency] = useState("USD");
  const [amount, setAmount] = useState("");
  const [conversion, setConversion] = useState({ from: "USD", to: "BTC", rate: 0 });

  useEffect(() => {
    // Fetch wallet data on load
    axios.get("/api/wallet").then((response) => {
      setBalance(response.data.balance);
      setCurrency(response.data.currency);
    });
  }, []);

  const handleDeposit = () => {
    axios
      .post("/api/wallet/deposit", { amount, currency })
      .then((response) => {
        setBalance(response.data.new_balance);
        setAmount("");
        alert("Deposit successful!");
      })
      .catch((error) => alert(error.response.data.message || "Error"));
  };

  const handleConvert = () => {
    axios
      .post("/api/wallet/convert", {
        from_currency: conversion.from,
        to_currency: conversion.to,
        amount,
      })
      .then((response) => {
        setBalance(response.data.new_balance);
        setAmount("");
        alert(`Converted to ${response.data.converted_amount} ${conversion.to}`);
      })
      .catch((error) => alert(error.response.data.message || "Error"));
  };

  const fetchConversionRate = () => {
    axios
      .get(`/api/conversion-rate?from=${conversion.from}&to=${conversion.to}`)
      .then((response) => {
        setConversion((prev) => ({ ...prev, rate: response.data.rate }));
      });
  };

  useEffect(() => {
    fetchConversionRate();
  }, [conversion.from, conversion.to]);

  return (
    <div className="p-4 flex flex-col gap-4 max-w-xl mx-auto">
      <Card>
        <CardContent className="text-center">
          <h1 className="text-xl font-bold">Digital Wallet</h1>
          <p className="mt-2">Balance: {balance} {currency}</p>
        </CardContent>
      </Card>

      <Card>
        <CardContent>
          <h2 className="text-lg font-semibold mb-2">Deposit</h2>
          <Input
            placeholder="Amount"
            value={amount}
            onChange={(e) => setAmount(e.target.value)}
            type="number"
          />
          <Button className="mt-2 w-full" onClick={handleDeposit}>
            Deposit
          </Button>
        </CardContent>
      </Card>

      <Card>
        <CardContent>
          <h2 className="text-lg font-semibold mb-2">Convert Currency</h2>
          <div className="flex gap-2">
            <Input
              placeholder="Amount"
              value={amount}
              onChange={(e) => setAmount(e.target.value)}
              type="number"
            />
            <Input
              placeholder="From Currency"
              value={conversion.from}
              onChange={(e) =>
                setConversion((prev) => ({ ...prev, from: e.target.value }))
              }
            />
            <Input
              placeholder="To Currency"
              value={conversion.to}
              onChange={(e) =>
                setConversion((prev) => ({ ...prev, to: e.target.value }))
              }
            />
          </div>
          <p className="mt-2 text-sm">Conversion Rate: 1 {conversion.from} = {conversion.rate} {conversion.to}</p>
          <Button className="mt-2 w-full" onClick={handleConvert}>
            Convert
          </Button>
        </CardContent>
      </Card>
    </div>
  );
};

export default WalletApp;
