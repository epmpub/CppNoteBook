# asio parse websites[[待修复]]

```C++
#include <boost/asio.hpp>
#include <boost/bind.hpp>
#include <chrono>
#include <fstream>
#include <iomanip>
#include <iostream>
#include <string>
#include <vector>
#include <map>
#include <algorithm>
#include <mutex>

using boost::asio::ip::tcp;

class TcpTester {
public:
	TcpTester() : io_context_(), resolver_(io_context_), active_connections_(0), max_connections_(12) {}

	void run(const std::vector<std::string>& urls) {
		urls_ = urls;
		results_.reserve(urls.size());

		// Pre-resolve DNS
		for (const auto& url : urls_) {
			resolve_dns(url);
		}
		io_context_.run();
	}

	const std::vector<std::string>& get_results() const {
		return results_;
	}

private:
	void resolve_dns(const std::string& url) {
		auto resolver = std::make_shared<tcp::resolver>(io_context_);
		resolver->async_resolve(url, "https",
			boost::bind(&TcpTester::handle_resolve, this, url, resolver,
				boost::asio::placeholders::error,
				boost::asio::placeholders::results));
	}

	void handle_resolve(const std::string& url, std::shared_ptr<tcp::resolver>,
		const boost::system::error_code& ec,
		tcp::resolver::results_type results) {
		if (!ec) {
			for (const auto& endpoint : results) {
				if (endpoint.endpoint().address().is_v4()) {
					std::lock_guard<std::mutex> lock(mutex_);
					dns_cache_[url] = endpoint.endpoint().address().to_string();
					break;
				}
			}
		}
		if (dns_cache_.find(url) == dns_cache_.end()) {
			std::lock_guard<std::mutex> lock(mutex_);
			dns_cache_[url] = url; // Fallback to URL
		}

		// Start TCP test if all DNS resolutions are done or immediately for this URL
		if (dns_cache_.size() == urls_.size()) {
			io_context_.reset();
			for (const auto& url : urls_) {
				test_connection(url, dns_cache_[url]);
			}
			io_context_.run();
		}
	}

	void test_connection(const std::string& url, const std::string& ip) {
		if (active_connections_ >= max_connections_) {
			pending_urls_.push_back({ url, ip });
			return;
		}

		++active_connections_;
		auto socket = std::make_shared<tcp::socket>(io_context_);
		tcp::endpoint endpoint(boost::asio::ip::address::from_string(ip), 443);

		// Set 5-second timeout
		auto timer = std::make_shared<boost::asio::steady_timer>(io_context_);
		timer->expires_after(std::chrono::seconds(5));

		socket->async_connect(endpoint,
			boost::bind(&TcpTester::handle_connect, this, url, socket, timer,
				boost::asio::placeholders::error));

		timer->async_wait(
			boost::bind(&TcpTester::handle_timeout, this, url, socket, timer,
				boost::asio::placeholders::error));
	}

	void handle_connect(const std::string& url, std::shared_ptr<tcp::socket> socket,
		std::shared_ptr<boost::asio::steady_timer> timer,
		const boost::system::error_code& ec) {
		if (!ec) {
			timer->cancel(); // Cancel timeout
			std::string result = std::string(url).append(20 - url.size(), ' ')
				+ std::string("443").append(10 - 3, ' ')
				+ "True";
			std::lock_guard<std::mutex> lock(mutex_);
			results_.push_back(result);
		}
		else if (ec != boost::asio::error::operation_aborted) {
			handle_failure(url);
		}

		complete_connection();
	}

	void handle_timeout(const std::string& url, std::shared_ptr<tcp::socket> socket,
		std::shared_ptr<boost::asio::steady_timer>,
		const boost::system::error_code& ec) {
		if (!ec) {
			socket->close(); // Timeout occurred
			handle_failure(url);
		}
		complete_connection();
	}

	void handle_failure(const std::string& url) {
		std::string result = std::string(url).append(20 - url.size(), ' ')
			+ std::string("443").append(10 - 3, ' ')
			+ "False";
		std::lock_guard<std::mutex> lock(mutex_);
		results_.push_back(result);
	}

	void complete_connection() {
		std::lock_guard<std::mutex> lock(mutex_);
		--active_connections_;

		// Start next pending connection
		if (!pending_urls_.empty() && active_connections_ < max_connections_) {
			auto [url, ip] = pending_urls_.back();
			pending_urls_.pop_back();
			test_connection(url, ip);
		}
	}

	boost::asio::io_context io_context_;
	tcp::resolver resolver_;
	std::vector<std::string> urls_;
	std::map<std::string, std::string> dns_cache_;
	std::vector<std::string> results_;
	std::vector<std::pair<std::string, std::string>> pending_urls_;
	size_t active_connections_;
	const size_t max_connections_;
	std::mutex mutex_;
};

int main() {
	// List of URLs
	std::vector<std::string> urls = {
		"google.com", "youtube.com", "facebook.com", "twitter.com", "instagram.com",
		"linkedin.com", "reddit.com", "pinterest.com", "tumblr.com", "x.com",
		"microsoft.com", "aws.com"
	};

	// Measure execution time
	auto start = std::chrono::high_resolution_clock::now();

	// Run TCP tests
	TcpTester tester;
	tester.run(urls);
	auto results = tester.get_results();

	// Sort results
	std::sort(results.begin(), results.end());

	// Calculate execution time
	auto end = std::chrono::high_resolution_clock::now();
	auto duration = std::chrono::duration_cast<std::chrono::nanoseconds>(end - start).count() / 1e9;

	// Write output to file
	std::ofstream file("testWebHostPwsh.txt");
	file << std::left << std::setw(20) << "ComputerName" << std::setw(12) << "RemotePort" << std::setw(15) << "TcpTestSucceeded" << "\n";
	file << "------------         ---------- ----------------\n";
	for (const auto& result : results) {
		file << result << "\n";
	}
	file << "\n";
	file << std::fixed << std::setprecision(7) << "Execution Time: " << duration << " seconds\n";
	file.close();

	return 0;
}
```

