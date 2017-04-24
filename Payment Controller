using PayPal.Api;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using System.Web.Mvc;
using static XXX.Data.Classes.XXXEnums;
using static XXX.Data.Models.OrderModels;
using static XXX.Data.Models.PaymentModels;
 
 public class PaymentController : Controller
    {
        private OrderServices OrderServices = new OrderServices();

        [HttpPost]
        [Authorize]
        [ValidateAntiForgeryToken]
        public ActionResult PayWithPayPal(string os_id)
        {
            try
            {
                if (!String.IsNullOrWhiteSpace(os_id))
                {
                    var _order = Session[os_id];

                    if (_order != null)
                    {
                        if (_order is OrderModel)
                        {
                            var order = _order as OrderModel;

                            var config = ConfigManager.Instance.GetProperties();

                            var accessToken = new OAuthTokenCredential(config).GetAccessToken();

                            var apiContext = new APIContext(accessToken);

                            var items = new List<Item>();

                            double orderAmount = 0;

                            foreach (var item in order.OrderDetails)
                            {
                                if (item != null)
                                {
                                    items.Add(new Item
                                    {
                                        price = item.UnitPrice.ToString(),
                                        currency = "USD",
                                        sku = item.Product.StockNumber.ToString(),
                                        name = item.Product.Title_English,
                                        quantity = item.Quantity.ToString()
                                    });

                                    orderAmount += item.UnitPrice * item.Quantity;
                                }
                            }

                            var itemList = new ItemList();

                            itemList.items = items;

                            var payer = new Payer() { payment_method = "paypal" };

                            var baseURI = Request.Url.Scheme + "://" + Request.Url.Authority + "/Payment/PaymentApprove?";

                            var ps_id = Guid.NewGuid().ToString();

                            var ks_token = Guid.NewGuid().ToString();

                            var pys_token = Guid.NewGuid().ToString();

                            var pyc_token = Guid.NewGuid().ToString();

                            var redirectCancelUrl = baseURI + "&os_id=" + os_id + "&ps_id=" + ps_id + "&ks_token=" + ks_token + "&pyc_token=" + pyc_token;

                            var redirectSuccessUrl = baseURI + "&os_id=" + os_id + "&ps_id=" + ps_id + "&ks_token=" + ks_token + "&pys_token=" + pys_token;

                            var redirUrls = new RedirectUrls()
                            {
                                cancel_url = redirectCancelUrl,
                                return_url = redirectSuccessUrl
                            };

                            var details = new Details()
                            {
                                tax = "0",
                                shipping = "0",
                                subtotal = orderAmount.ToString()
                            };

                            var amount = new Amount()
                            {
                                currency = "USD",
                                total = orderAmount.ToString(),
                                details = details
                            };

                            var transactionList = new List<Transaction>();

                            transactionList.Add(new Transaction()
                            {
                                description = "Transaction description.",
                                invoice_number = "125",
                                amount = amount,
                                item_list = itemList
                            });

                            var payment = new Payment()
                            {
                                intent = "sale",
                                payer = payer,
                                transactions = transactionList,
                                redirect_urls = redirUrls
                            };

                            var createdPayment = payment.Create(apiContext);

                            var links = createdPayment.links.GetEnumerator();

                            var approvalUrl = createdPayment.links.Single(l => l.rel == "approval_url").href;

                            var paymentApproveData = new PayPalPaymentApproveModel()
                            {
                                OS_ID = os_id,
                                PS_ID = ps_id,
                                PYS_TOKEN = pys_token,
                                PYC_TOKEN = pyc_token,
                                KS_TOKEN = ks_token,
                                PayerId = createdPayment.payer.funding_option_id,
                                PaymentId = createdPayment.id,
                                Token = createdPayment.token,
                            };

                            Session[ps_id] = paymentApproveData;

                            var paymentId = createdPayment.id;

                            return Redirect(approvalUrl);

                        }
                    }
                }

                return RedirectToAction("Cart", "Profile");
            }
            catch (Exception)
            {
                return RedirectToAction("Cart", "Profile");
            }

        }


        public async Task<ActionResult> PaymentApprove(PayPalPaymentApproveModel model)
        {
            try
            {
                if (ModelState.IsValid)
                {
                    if (!String.IsNullOrWhiteSpace(model.PS_ID))
                    {
                        var _paymentApproveData = Session[model.PS_ID];

                        var _order = Session[model.OS_ID];

                        if (_paymentApproveData != null && _paymentApproveData is PayPalPaymentApproveModel && _order != null && _order is OrderModel)
                        {
                            var paymentApproveData = _paymentApproveData as PayPalPaymentApproveModel;

                            var order = _order as OrderModel;

                            if (model.PYC_TOKEN == paymentApproveData.PYC_TOKEN)
                            {
                                return View("PaymentCancel");
                            }

                            if (model.KS_TOKEN == paymentApproveData.KS_TOKEN
                                && model.PYS_TOKEN == paymentApproveData.PYS_TOKEN
                                && model.PaymentId == paymentApproveData.PaymentId
                                && model.Token == paymentApproveData.Token
                                )
                            {
                                var result = await OrderServices.AddAync(order);
                               
                                return View("PaymentSuccess");
                            }

                        }
                    }
                }
            }

            catch (Exception)
            {
                return HttpNotFound();
            }

            return HttpNotFound();
        }
        
        protected override void Dispose(bool disposing)
        {
            if (disposing)
            {
                OrderServices.Dispose();
            }

            base.Dispose(disposing);
        }
