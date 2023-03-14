## This code creates a custom menu bar item using the NSStatusItem class and updates its title every minute with the time left for the day, quarter, and year. You can customize the code to fit your specific needs and integrate it into your Mac desktop app.


``` import Cocoa

class StatusBarController {
    let statusItem = NSStatusBar.system.statusItem(withLength: NSStatusItem.variableLength)
    
    func updateStatusBar() {
        let calendar = Calendar.current
        let now = Date()
        
        // Calculate time left for the day
        let endOfDay = calendar.startOfDay(for: now).addingTimeInterval(24 * 60 * 60)
        let timeLeftToday = endOfDay.timeIntervalSince(now) / 3600
        
        // Calculate time left for the quarter
        let startOfQuarter = calendar.date(from: DateComponents(month: (calendar.component(.month, from: now) - 1) / 3 * 3, day: 1))!
        let endOfQuarter = calendar.date(byAdding: DateComponents(month: 3, day: -1), to: startOfQuarter)!
        let timeLeftQuarter = endOfQuarter.timeIntervalSince(now) / (3600 * 24 * 7)
        
        // Calculate time left for the year
        let startOfYear = calendar.date(from: DateComponents(year: calendar.component(.year, from: now), month: 1, day: 1))!
        let endOfYear = calendar.date(byAdding: DateComponents(year: 1, day: -1), to: startOfYear)!
        let timeLeftYear = endOfYear.timeIntervalSince(now) / (3600 * 24 * 7)
        
        // Update status bar item title
        statusItem.button?.title = String(format: "%.0f Hours (Today), %.0f Weeks (Q%d), %.0f Weeks (%d)", timeLeftToday, timeLeftQuarter, (calendar.component(.month, from: now) - 1) / 3 + 1, timeLeftYear, calendar.component(.year, from: now))
    }
    
    init() {
        updateStatusBar()
        Timer.scheduledTimer(withTimeInterval: 60, repeats: true) { _ in
            self.updateStatusBar()
        }
    }
}

let statusBarController = StatusBarController()
RunLoop.main.run()
``` 
 
## This code creates a Mac desktop app that replaces the clock in the top right bar with the number of hours left in the day, number of weeks left in the quarter, and number of weeks left in the year. The code uses the Cocoa framework to create a status bar item and menu, and the Swift Date and Calendar APIs to perform date and time calculations. The app updates the menu item title every minute using the DispatchQueue API to run the updateLabel function on the main thread asynchronously.
``` import Cocoa

func calculateTimeLeft() -> (Int, Int, Int) {
    let now = Date()
    let calendar = Calendar.current
    
    // Calculate time left for the day
    let endOfDay = calendar.date(bySettingHour: 23, minute: 59, second: 59, of: now)!
    let timeLeftToday = calendar.dateComponents([.hour], from: now, to: endOfDay).hour!
    
    // Calculate time left for the quarter
    let currentMonth = calendar.component(.month, from: now)
    let currentQuarter = Int(ceil(Double(currentMonth) / 3.0))
    let startOfQuarter = calendar.date(from: DateComponents(year: calendar.component(.year, from: now), month: (currentQuarter - 1) * 3 + 1, day: 1))!
    let endOfQuarter = calendar.date(byAdding: DateComponents(month: 3, day: 31), to: startOfQuarter)!
    let timeLeftQuarter = calendar.dateComponents([.weekOfYear], from: now, to: endOfQuarter).weekOfYear!
    
    // Calculate time left for the year
    let startOfYear = calendar.date(from: DateComponents(year: calendar.component(.year, from: now), month: 1, day: 1))!
    let endOfYear = calendar.date(bySetting: .month, value: 12, of: startOfYear)!
    let timeLeftYear = calendar.dateComponents([.weekOfYear], from: now, to: endOfYear).weekOfYear!
    
    return (timeLeftToday, timeLeftQuarter, timeLeftYear)
}

let statusItem = NSStatusBar.system.statusItem(withLength: NSStatusItem.variableLength)
let menu = NSMenu()
let menuItem = NSMenuItem(title: "", action: nil, keyEquivalent: "")
menu.addItem(menuItem)
statusItem.menu = menu

func updateLabel() {
    let (timeLeftToday, timeLeftQuarter, timeLeftYear) = calculateTimeLeft()
    menuItem.title = "\(timeLeftToday) Hours (Today), \(timeLeftQuarter) Weeks (Q\(Int(ceil(Double(Calendar.current.component(.month, from: Date())) / 3.0)))), \(timeLeftYear) Weeks (\(Calendar.current.component(.year, from: Date())))"
    DispatchQueue.main.asyncAfter(deadline: .now() + 60.0, execute: updateLabel) // update every minute
}

updateLabel()

RunLoop.main.run()
```




